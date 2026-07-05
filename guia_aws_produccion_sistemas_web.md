# Guía de AWS para producción de sistemas web

## Objetivo

Esta guía resume buenas prácticas para montar un entorno productivo de sistemas web en AWS. Está acotada a una arquitectura clásica y robusta para aplicaciones web con frontend, backend, base de datos, almacenamiento, autenticación, mensajería, observabilidad e infraestructura como código.

La guía está organizada en tres niveles:

```text
1. Básico: conceptos, regiones, VPC, subredes, gateways, security groups, EC2, S3 y RDS.
2. Intermedio: ALB, Auto Scaling, NAT Gateway, CloudFront, Cognito, IAM, Secrets Manager, Parameter Store, SNS, SQS y Lambda.
3. Avanzado: EventBridge, CloudFormation, CloudWatch, CloudTrail, X-Ray, arquitectura multi-AZ, operación, seguridad, costos y checklists productivas.
```

El foco es una arquitectura de producción razonable para sistemas web, no todos los servicios de AWS.

---

# Parte I: arquitectura objetivo

## 1. Arquitectura de referencia

Arquitectura propuesta:

```text
Internet
   │
   ▼
Route 53 / DNS
   │
   ▼
CloudFront ───────────────► S3 privado para frontend estático
   │
   ▼
Application Load Balancer público
   │
   ▼
EC2 Auto Scaling Group en subredes privadas
   │
   ├────────► RDS privado en subredes privadas
   │
   ├────────► S3 privado para uploads y archivos
   │
   ├────────► Secrets Manager / Parameter Store
   │
   ├────────► SQS / SNS / EventBridge
   │
   └────────► CloudWatch Logs / X-Ray

CloudTrail registra acciones de cuenta.
IAM controla permisos.
Cognito gestiona autenticación de usuarios.
```

Principios:

```text
El tráfico público entra por CloudFront o ALB.
Las instancias de aplicación viven en subredes privadas.
La base de datos no es pública.
Los secretos no viven en código.
Los logs y métricas se centralizan.
La infraestructura se define con CloudFormation o IaC equivalente.
```

---

## 2. Componentes principales

| Componente | Rol en producción web |
|---|---|
| VPC | Red aislada donde viven recursos |
| Subred pública | Aloja ALB, NAT Gateway y recursos expuestos controladamente |
| Subred privada | Aloja EC2 backend, RDS, workers y servicios internos |
| Internet Gateway | Permite tráfico entre VPC e internet para subredes públicas |
| NAT Gateway | Permite salida a internet desde subredes privadas sin aceptar entrada directa |
| ALB | Balancea tráfico HTTP/HTTPS hacia backends |
| EC2 | Ejecuta aplicación web/backend |
| Auto Scaling Group | Mantiene y escala instancias EC2 |
| RDS | Base de datos gestionada |
| S3 | Archivos estáticos, uploads, backups, logs |
| CloudFront | CDN, TLS, caching y protección de origen |
| Cognito | Autenticación de usuarios |
| IAM | Permisos entre usuarios, roles y servicios |
| Secrets Manager | Secretos rotables como credenciales |
| Parameter Store | Configuración y parámetros jerárquicos |
| SQS | Cola de mensajes |
| SNS | Pub/sub y notificaciones |
| Lambda | Cómputo serverless por eventos |
| EventBridge | Bus de eventos y programación |
| CloudFormation | Infraestructura como código |
| CloudWatch | Métricas, logs, alarmas y dashboards |
| CloudTrail | Auditoría de acciones de cuenta |
| X-Ray | Trazas distribuidas |

---

## 3. Diseño multi-AZ

Producción debe usar al menos dos Availability Zones.

Ejemplo:

```text
VPC: 10.0.0.0/16

AZ A:
- Public subnet A:  10.0.1.0/24
- Private app A:    10.0.11.0/24
- Private data A:   10.0.21.0/24

AZ B:
- Public subnet B:  10.0.2.0/24
- Private app B:    10.0.12.0/24
- Private data B:   10.0.22.0/24
```

Separación recomendada:

```text
Subredes públicas:
ALB, NAT Gateway, bastion solo si es estrictamente necesario.

Subredes privadas de aplicación:
EC2, ECS, workers, Lambdas con VPC si aplica.

Subredes privadas de datos:
RDS, ElastiCache, bases internas.
```

Regla:

```text
Que una subred sea pública o privada depende de su tabla de rutas,
no de su nombre.
```

---

# Parte II: nivel básico

## 4. Regiones y Availability Zones

Una región es una ubicación geográfica de AWS.

Ejemplos:

```text
us-east-1
us-east-2
sa-east-1
eu-west-1
```

Una Availability Zone es un datacenter o grupo de datacenters aislado dentro de una región.

Buenas prácticas:

```text
Elige una región cercana a tus usuarios.
Usa al menos dos AZ para producción.
Verifica disponibilidad de servicios en la región.
Considera requisitos legales de residencia de datos.
Considera costo y latencia.
```

---

## 5. Crear una VPC

Una VPC define el espacio de red de tu aplicación.

Ejemplo:

```text
Nombre: web-prod-vpc
CIDR:   10.0.0.0/16
```

Buenas prácticas:

```text
Usa un CIDR amplio pero planificado.
Evita solapamiento con redes corporativas o VPN.
Reserva rangos por ambiente.
No uses la default VPC para producción seria.
```

Ejemplo de distribución:

```text
dev:     10.10.0.0/16
staging: 10.20.0.0/16
prod:    10.30.0.0/16
```

---

## 6. Crear subredes públicas

Una subred pública tiene una ruta directa al Internet Gateway.

Ejemplo:

```text
public-a: 10.0.1.0/24
public-b: 10.0.2.0/24
```

Recursos típicos:

```text
Application Load Balancer.
NAT Gateway.
Bastion host, solo si es necesario.
```

No pongas aquí:

```text
Base de datos.
Backends internos.
Workers.
Servicios con secretos.
```

---

## 7. Crear subredes privadas

Una subred privada no tiene ruta directa al Internet Gateway.

Ejemplo:

```text
private-app-a:  10.0.11.0/24
private-app-b:  10.0.12.0/24
private-data-a: 10.0.21.0/24
private-data-b: 10.0.22.0/24
```

Recursos típicos:

```text
EC2 backend.
Workers.
RDS.
ElastiCache.
Servicios internos.
```

Salida a internet:

```text
Las subredes privadas pueden salir por NAT Gateway,
pero no reciben conexiones entrantes desde internet.
```

---

## 8. Internet Gateway

Un Internet Gateway permite que recursos en subredes públicas se comuniquen con internet.

Pasos:

```text
1. Crear Internet Gateway.
2. Asociarlo a la VPC.
3. Crear ruta 0.0.0.0/0 hacia el Internet Gateway en tabla de rutas pública.
4. Asociar subredes públicas a esa tabla.
```

Tabla pública:

```text
Destino       Target
10.0.0.0/16   local
0.0.0.0/0     igw-xxxxxxxx
```

---

## 9. NAT Gateway

Un NAT Gateway permite que instancias en subredes privadas hagan requests salientes a internet.

Uso típico:

```text
Instalar paquetes.
Descargar imágenes.
Llamar APIs externas.
Contactar servicios públicos.
```

Pasos:

```text
1. Crear Elastic IP.
2. Crear NAT Gateway en subred pública.
3. Asociar Elastic IP.
4. Agregar ruta 0.0.0.0/0 hacia NAT Gateway en tabla privada.
```

Tabla privada:

```text
Destino       Target
10.0.0.0/16   local
0.0.0.0/0     nat-xxxxxxxx
```

Buenas prácticas:

```text
Crea un NAT Gateway por AZ para alta disponibilidad.
Las subredes privadas de cada AZ deberían salir por el NAT de su misma AZ.
Evalúa costos: NAT Gateway cobra por hora y por datos procesados.
Usa VPC endpoints para reducir tráfico por NAT hacia servicios AWS como S3.
```

Arquitectura mínima de bajo costo:

```text
Un NAT Gateway compartido.
```

Arquitectura productiva más robusta:

```text
Un NAT Gateway por AZ.
```

---

## 10. VPC endpoints

Un VPC endpoint permite acceder a servicios AWS sin salir por internet.

Tipos:

```text
Gateway endpoint:
S3 y DynamoDB.

Interface endpoint:
Secrets Manager, SQS, SNS, CloudWatch Logs, ECR, Systems Manager y otros.
```

Uso recomendado:

```text
S3 Gateway Endpoint para que EC2 privadas accedan a S3 sin NAT.
Interface Endpoints para Secrets Manager, SQS, CloudWatch Logs o Systems Manager si buscas tráfico privado y menor dependencia del NAT.
```

Ejemplo:

```text
Private EC2 -> S3 Gateway Endpoint -> S3
```

Ventajas:

```text
Menos tráfico por NAT.
Mejor seguridad de red.
Menor exposición.
Puede reducir costos según patrón de uso.
```

---

## 11. Route tables

Cada subred se asocia a una tabla de rutas.

Patrón:

```text
Public route table:
- local
- 0.0.0.0/0 -> Internet Gateway

Private app route table A:
- local
- 0.0.0.0/0 -> NAT Gateway A
- S3 prefix list -> S3 Gateway Endpoint

Private app route table B:
- local
- 0.0.0.0/0 -> NAT Gateway B
- S3 prefix list -> S3 Gateway Endpoint

Private data route table:
- local
- normalmente sin ruta 0.0.0.0/0 si no necesita salida
```

Regla:

```text
Las subredes de datos deben tener la mínima conectividad necesaria.
```

---

## 12. Security Groups

Un Security Group actúa como firewall a nivel de recurso.

Ejemplo de grupos:

```text
sg-alb-public
sg-app-private
sg-rds-private
sg-vpc-endpoints
```

Reglas recomendadas:

### ALB

Inbound:

```text
HTTPS 443 desde 0.0.0.0/0
HTTP 80 desde 0.0.0.0/0 solo para redirección a HTTPS
```

Outbound:

```text
Puerto app 8000 hacia sg-app-private
```

### App EC2

Inbound:

```text
Puerto app 8000 desde sg-alb-public
```

Outbound:

```text
PostgreSQL 5432 hacia sg-rds-private
HTTPS 443 hacia internet o VPC endpoints
```

### RDS

Inbound:

```text
PostgreSQL 5432 desde sg-app-private
```

Outbound:

```text
Normalmente no necesario o muy restringido
```

Regla:

```text
Referencia security groups entre sí.
No abras RDS a 0.0.0.0/0.
No abras SSH a internet en producción.
```

---

## 13. Network ACLs

Network ACLs son reglas stateless a nivel de subred.

Recomendación práctica:

```text
Usa Security Groups como control principal.
Mantén NACLs simples salvo requisitos específicos.
Usa NACLs para bloqueos gruesos, no para lógica fina de aplicación.
```

---

# Parte III: cómputo web con EC2, ALB y Auto Scaling

## 14. Levantar instancias EC2

EC2 ejecuta servidores virtuales.

Configuración base:

```text
AMI: Amazon Linux 2023, Ubuntu LTS u otra imagen aprobada.
Instance type: t3/t4g para cargas pequeñas, familia m/c/r según necesidad.
Subnet: privada de aplicación.
Security group: sg-app-private.
IAM role: rol de instancia con permisos mínimos.
Storage: EBS gp3 cifrado.
User data: script de bootstrap.
```

No recomendado en producción:

```text
Instancia EC2 única pública.
SSH abierto a internet.
Credenciales dentro de archivos.
Deploy manual sin reproducibilidad.
```

---

## 15. Acceso a instancias privadas

Opciones:

```text
AWS Systems Manager Session Manager.
Bastion host.
VPN / Direct Connect.
```

Recomendación:

```text
Prefiere Systems Manager Session Manager para evitar abrir SSH.
```

Necesitas:

```text
IAM role en la instancia.
SSM Agent instalado.
Conectividad a Systems Manager mediante internet/NAT o VPC endpoints.
```

---

## 16. User data básico

Ejemplo para instalar una app simple:

```bash
#!/bin/bash
set -euo pipefail

dnf update -y
dnf install -y python3 python3-pip git

useradd --system --create-home appuser || true

cd /opt
git clone https://github.com/example/my-app.git
cd my-app

python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

cat > /etc/systemd/system/my-app.service <<'EOF'
[Unit]
Description=My Web App
After=network.target

[Service]
User=appuser
WorkingDirectory=/opt/my-app
Environment=APP_ENV=production
ExecStart=/opt/my-app/.venv/bin/uvicorn app.main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable my-app
systemctl start my-app
```

En producción real, evita depender de `git clone` en cada instancia. Mejor:

```text
AMI preconstruida.
Artefacto versionado en S3.
Docker/ECS.
CodeDeploy.
Pipeline de despliegue.
```

---

## 17. Application Load Balancer

Un ALB distribuye tráfico HTTP/HTTPS entre targets.

Componentes:

```text
Load balancer.
Listeners.
Rules.
Target groups.
Health checks.
Security group.
```

Patrón:

```text
Internet -> ALB público -> Target group -> EC2 privadas
```

Listeners:

```text
80  -> redirect a 443
443 -> forward a target group app
```

Target group:

```text
Protocol: HTTP
Port: 8000
Health check path: /health
Matcher: 200
Targets: EC2 del Auto Scaling Group
```

Health check recomendado:

```http
GET /health
```

Respuesta:

```json
{
    "status": "ok"
}
```

Regla:

```text
El ALB solo debería enviar tráfico a instancias saludables.
```

---

## 18. Auto Scaling Group

Un Auto Scaling Group mantiene la cantidad deseada de instancias.

Configuración:

```text
Min capacity: 2
Desired capacity: 2
Max capacity: 6
Subnets: private-app-a, private-app-b
Launch template: define AMI, tipo, SG, IAM role, user data
Target group: app target group del ALB
Health checks: EC2 + ELB
```

Políticas de escalamiento:

```text
CPUUtilization promedio > 60%
RequestCountPerTarget
ALB TargetResponseTime
SQS queue depth para workers
```

Buenas prácticas:

```text
Usa al menos 2 instancias en 2 AZ para producción.
Activa health checks del ALB.
Diseña la app stateless.
Guarda sesiones fuera de EC2: cookies firmadas, Redis, DB o Cognito.
No guardes archivos persistentes en disco local.
```

---

## 19. Blue/Green y rolling deployments

Opciones:

```text
Rolling:
Actualizar instancias gradualmente.

Blue/Green:
Levantar entorno nuevo y cambiar tráfico.

Canary:
Enviar bajo porcentaje al nuevo release.

Feature flags:
Activar funcionalidad por porcentaje, tenant o usuario.
```

Para EC2 puedes usar:

```text
CodeDeploy.
Auto Scaling rolling update.
CloudFormation update policies.
ALB weighted target groups.
```

Checklist de despliegue:

```text
[ ] Build generado.
[ ] Tests pasan.
[ ] Imagen/AMI/artefacto versionado.
[ ] Migraciones probadas.
[ ] Health checks OK.
[ ] Logs sin errores nuevos.
[ ] Rollback definido.
```

---

# Parte IV: bases de datos con RDS

## 20. RDS en producción

RDS gestiona bases relacionales como PostgreSQL, MySQL, MariaDB, SQL Server y Oracle.

Patrón para producción:

```text
RDS en subredes privadas de datos.
DB subnet group con al menos dos AZ.
Security group que solo acepte conexiones desde app.
Backups automáticos.
Cifrado.
Multi-AZ para alta disponibilidad.
Monitoring y Performance Insights si aplica.
```

---

## 21. Crear DB Subnet Group

Un DB subnet group define en qué subredes puede vivir RDS.

Incluye:

```text
private-data-a
private-data-b
```

No incluyas subredes públicas salvo una razón técnica explícita.

---

## 22. Crear RDS PostgreSQL

Configuración base:

```text
Engine: PostgreSQL.
Deployment: Multi-AZ para producción.
Public access: No.
DB subnet group: privado.
Security group: sg-rds-private.
Storage: gp3 o io1/io2 según carga.
Storage encryption: enabled.
Backup retention: 7-35 días según RPO.
Deletion protection: enabled.
Master password: Secrets Manager.
```

Buenas prácticas:

```text
No uses usuario master desde la aplicación.
Crea usuario app con permisos mínimos.
Activa backups automáticos.
Prueba restauraciones.
Usa migraciones versionadas.
Monitorea conexiones y queries lentas.
```

---

## 23. Seguridad de RDS

Reglas:

```text
RDS sin acceso público.
Puerto 5432 solo desde sg-app-private.
Credenciales en Secrets Manager.
Cifrado en reposo.
TLS para conexiones si aplica.
Backups cifrados.
Logs exportados a CloudWatch cuando corresponda.
```

Ejemplo de Security Group:

```text
sg-rds-private inbound:
- PostgreSQL 5432 from sg-app-private
```

---

## 24. Conexiones y pooling

Problema común:

```text
Muchas instancias o Lambdas abren demasiadas conexiones a RDS.
```

Soluciones:

```text
Pool de conexiones en la app.
RDS Proxy para ciertos casos.
Limitar workers.
Usar PgBouncer si administras el patrón.
```

Para aplicaciones Python:

```text
SQLAlchemy engine pool.
asyncpg pool.
psycopg_pool.
```

---

## 25. Backups y restauración

Configura:

```text
Automated backups.
Backup retention.
Snapshots manuales antes de cambios grandes.
PITR si necesitas recuperación a punto en el tiempo.
Pruebas de restore.
```

Regla:

```text
Un backup no probado no es una garantía.
```

---

# Parte V: S3 y CloudFront

## 26. S3 para frontend estático

Para una SPA o frontend estático:

```text
Build React/Next estático -> S3 privado -> CloudFront -> usuarios
```

Buenas prácticas:

```text
Mantén Block Public Access activado.
No hagas público el bucket si usarás CloudFront.
Usa CloudFront Origin Access Control.
Versiona assets con hashes.
Configura cache headers.
Habilita logs si necesitas auditoría.
```

---

## 27. S3 para uploads

Patrón:

```text
Frontend solicita URL firmada al backend.
Backend valida usuario y genera presigned URL.
Frontend sube directamente a S3.
Backend guarda metadata en DB.
```

Ventajas:

```text
Evita pasar archivos grandes por backend.
Permite controlar permisos.
Reduce carga de EC2.
```

Buenas prácticas:

```text
Buckets privados.
Objetos con nombres no predecibles.
Validar tipo y tamaño.
Antivirus o procesamiento si aplica.
Cifrado SSE-S3 o SSE-KMS.
Lifecycle policies.
No permitir listar bucket públicamente.
```

---

## 28. CloudFront

CloudFront actúa como CDN y capa de distribución global.

Usos:

```text
Servir frontend desde S3.
Cachear assets.
Terminar TLS.
Proteger origen con OAC.
Distribuir contenido globalmente.
Aplicar WAF si corresponde.
```

Orígenes típicos:

```text
S3 privado para frontend/assets.
ALB para API/backend.
```

Buenas prácticas:

```text
Usa OAC para S3.
Usa certificados ACM.
Redirige HTTP a HTTPS.
Define políticas de cache por path.
No cachees respuestas privadas.
Usa invalidaciones con criterio.
```

Ejemplo de paths:

```text
/assets/*  cache largo
/index.html cache corto
/api/*     no cache o cache muy controlado
```

---

## 29. CloudFront + ALB

Patrón:

```text
Usuario -> CloudFront -> ALB -> EC2 privadas
```

Ventajas:

```text
TLS y edge global.
WAF en CloudFront.
Header forwarding controlado.
Cache para contenido público.
Un solo dominio para frontend y API.
```

Precauciones:

```text
Configura origin request policy correctamente.
No cachees respuestas con Authorization si no corresponde.
Asegura que el ALB solo acepte tráfico esperado si necesitas restringir origen.
```

---

# Parte VI: identidad, permisos y secretos

## 30. IAM

IAM controla autenticación y autorización de usuarios, roles y servicios AWS.

Conceptos:

```text
User: identidad individual, evitar para workloads.
Group: agrupación de users.
Role: identidad asumible con permisos temporales.
Policy: documento JSON de permisos.
Principal: quién ejecuta.
Action: qué puede hacer.
Resource: sobre qué recurso.
Condition: bajo qué condiciones.
```

Buenas prácticas:

```text
Usa IAM Identity Center para humanos cuando sea posible.
Usa roles para workloads.
Aplica mínimo privilegio.
Evita access keys permanentes.
Activa MFA para humanos.
No uses root account para tareas diarias.
Usa IAM Access Analyzer.
Separa cuentas por ambiente o unidad de negocio si aplica.
```

---

## 31. Rol de EC2

La app en EC2 no debería tener claves en disco. Debe usar IAM Role.

Permisos típicos:

```text
Leer parámetros de SSM.
Leer secretos específicos de Secrets Manager.
Escribir logs en CloudWatch.
Leer/escribir un bucket S3 específico.
Enviar mensajes a una cola SQS específica.
Publicar a un topic SNS específico.
Enviar trazas X-Ray.
```

Ejemplo conceptual de política:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "secretsmanager:GetSecretValue"
            ],
            "Resource": "arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/db/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::my-app-prod-uploads/*"
        }
    ]
}
```

---

## 32. AWS Secrets Manager

Usa Secrets Manager para:

```text
Credenciales de base de datos.
API keys sensibles.
Secretos que requieren rotación.
Tokens de proveedores.
```

Buenas prácticas:

```text
Rotación si aplica.
Cifrado con KMS.
Políticas por secreto.
No leer secretos en cada request; cachear con TTL.
No imprimir secretos en logs.
Separar secretos por ambiente.
```

Ejemplo de nombres:

```text
/prod/web/db
/prod/web/jwt
/prod/payments/stripe
```

---

## 33. Systems Manager Parameter Store

Parameter Store sirve para configuración jerárquica.

Usa Parameter Store para:

```text
APP_ENV.
LOG_LEVEL.
Feature flags simples.
Endpoints internos.
Nombres de buckets.
Tamaños de batch.
Timeouts.
Configuración no secreta o secretos de menor complejidad.
```

Tipos:

```text
String.
StringList.
SecureString.
```

Ejemplo:

```text
/prod/web/app_env = production
/prod/web/log_level = INFO
/prod/web/uploads_bucket = my-app-prod-uploads
```

Diferencia práctica:

```text
Secrets Manager:
Secretos sensibles, rotación, credenciales.

Parameter Store:
Configuración jerárquica, parámetros, valores operativos.
```

---

## 34. Cognito

Cognito User Pools permite autenticación de usuarios para aplicaciones web y móviles.

Usos:

```text
Registro.
Login.
MFA.
Recuperación de contraseña.
Federación con Google, Apple, SAML u OIDC.
Hosted UI.
JWT tokens.
```

Componentes:

```text
User Pool:
Directorio de usuarios.

App Client:
Configuración de una aplicación que usa el user pool.

Domain:
Dominio para hosted UI.

Callback URLs:
Dónde vuelve el usuario después de login.

Logout URLs:
Dónde vuelve después de logout.
```

Buenas prácticas:

```text
Usa Authorization Code Flow con PKCE para SPAs.
Configura callback/logout URLs exactas.
Activa MFA para usuarios sensibles.
Valida tokens en backend.
No confíes solo en frontend.
Usa grupos/scopes si necesitas autorización.
```

Arquitectura:

```text
Frontend -> Cognito Hosted UI -> Callback frontend
Frontend -> API con access token
Backend -> valida JWT de Cognito
```

---

# Parte VII: mensajería, eventos y serverless

## 35. SQS

SQS es una cola. Un productor envía mensajes y uno o más consumidores los procesan.

Usos:

```text
Procesamiento asíncrono.
Emails.
Generación de reportes.
Procesamiento de imágenes.
Trabajos que no deben bloquear request HTTP.
Reintentos controlados.
Desacoplar servicios.
```

Patrón:

```text
API -> SQS -> Worker EC2/Lambda -> DB/S3
```

Buenas prácticas:

```text
Configura Dead Letter Queue.
Define visibility timeout mayor que el tiempo de procesamiento.
Haz consumidores idempotentes.
No asumas orden salvo FIFO queue.
Monitorea ApproximateAgeOfOldestMessage.
```

---

## 36. SNS

SNS es pub/sub. Publicas un mensaje a un topic y múltiples suscriptores lo reciben.

Usos:

```text
Notificaciones.
Fan-out a varias colas SQS.
Alertas.
Integraciones simples.
```

Patrón fan-out:

```text
API -> SNS topic
      ├── SQS email-queue
      ├── SQS analytics-queue
      └── Lambda audit-function
```

Buenas prácticas:

```text
Usa filtros de suscripción cuando corresponda.
No acoples payloads sin versionado.
Controla permisos de publicación.
```

---

## 37. Lambda

Lambda ejecuta código sin administrar servidores.

Usos:

```text
Procesar eventos S3.
Consumir SQS.
Responder a EventBridge.
Tareas programadas.
Webhooks simples.
Procesamiento liviano.
```

Buenas prácticas:

```text
Timeout explícito.
Memoria ajustada.
Variables por ambiente.
IAM role mínimo.
DLQ o destinos para errores asíncronos.
Idempotencia.
No guardar estado local.
Monitorear errores y duración.
```

Ejemplo conceptual en Python:

```python
import json


def handler(event, context):
    for record in event["Records"]:
        body = json.loads(record["body"])
        process_job(body)

    return {
        "statusCode": 200,
    }
```

---

## 38. EventBridge

EventBridge es un bus de eventos serverless.

Usos:

```text
Arquitectura event-driven.
Eventos de dominio.
Integraciones SaaS.
Programación de tareas.
Desacoplar productores y consumidores.
```

Ejemplo de evento:

```json
{
    "source": "myapp.tasks",
    "detail-type": "TaskCompleted",
    "detail": {
        "taskId": "task_123",
        "userId": "user_456"
    }
}
```

Patrón:

```text
Servicio A emite evento -> EventBridge -> reglas -> Lambda/SQS/SNS/Step Functions
```

Buenas prácticas:

```text
Versiona eventos.
Define schema.
Evita eventos enormes.
No incluyas secretos.
Diseña consumidores idempotentes.
```

---

## 39. SNS vs SQS vs EventBridge

| Servicio | Modelo | Uso principal |
|---|---|---|
| SQS | Cola pull | Trabajos asíncronos y desacoplamiento |
| SNS | Pub/sub push | Fan-out y notificaciones |
| EventBridge | Bus de eventos | Integración event-driven y ruteo por reglas |

Regla práctica:

```text
Usa SQS cuando necesitas cola y control del consumidor.
Usa SNS cuando quieres publicar a varios suscriptores.
Usa EventBridge cuando quieres arquitectura de eventos con reglas, fuentes y destinos.
```

---

# Parte VIII: observabilidad y auditoría

## 40. CloudWatch Logs

Usa CloudWatch Logs para centralizar logs de:

```text
EC2.
Lambda.
RDS logs.
ALB access logs, vía S3/Athena normalmente.
Aplicaciones.
```

Buenas prácticas:

```text
Logs JSON estructurados.
Retention definida.
No loguear secretos.
Correlation ID.
Filtros para errores.
Alarmas sobre patrones críticos.
```

Ejemplo de log JSON:

```json
{
    "level": "INFO",
    "event": "request_completed",
    "request_id": "req_123",
    "path": "/tasks",
    "status": 200,
    "duration_ms": 42
}
```

---

## 41. CloudWatch Metrics y Alarms

Monitorea:

```text
ALB:
- HTTPCode_ELB_5XX_Count
- HTTPCode_Target_5XX_Count
- TargetResponseTime
- HealthyHostCount

EC2:
- CPUUtilization
- Memory, con CloudWatch Agent
- Disk, con CloudWatch Agent

RDS:
- CPUUtilization
- DatabaseConnections
- FreeStorageSpace
- ReadLatency
- WriteLatency
- ReplicaLag si aplica

SQS:
- ApproximateAgeOfOldestMessage
- ApproximateNumberOfMessagesVisible

Lambda:
- Errors
- Duration
- Throttles
```

Alarmas mínimas:

```text
ALB 5xx alto.
Target 5xx alto.
Latencia p95 alta.
HealthyHostCount bajo.
RDS storage bajo.
RDS conexiones altas.
SQS mensajes viejos.
Lambda errors/throttles.
```

---

## 42. CloudWatch Dashboards

Dashboard recomendado:

```text
Tráfico:
Request count, status codes, latencia.

Aplicación:
Errores, logs, throughput, workers.

Base de datos:
CPU, conexiones, latencia, almacenamiento.

Colas:
Mensajes visibles, mensajes viejos, DLQ.

Infra:
CPU, memoria, disco, instancias saludables.
```

Regla:

```text
Un dashboard productivo debe responder si el sistema está sano y dónde está fallando.
```

---

## 43. CloudTrail

CloudTrail registra acciones realizadas por usuarios, roles y servicios.

Sirve para:

```text
Auditoría.
Investigación de incidentes.
Cumplimiento.
Detección de cambios sospechosos.
Historial de llamadas API.
```

Buenas prácticas:

```text
Crear trail multi-region.
Guardar logs en S3 dedicado y protegido.
Activar log file validation.
Enviar eventos a CloudWatch Logs si necesitas alertas.
Registrar management events.
Activar data events para S3/Lambda críticos cuando sea necesario.
```

Eventos a vigilar:

```text
CreateAccessKey.
AttachRolePolicy.
PutBucketPolicy.
AuthorizeSecurityGroupIngress.
DeleteTrail.
StopLogging.
RunInstances.
CreateUser.
```

---

## 44. X-Ray

X-Ray permite trazas distribuidas.

Útil para responder:

```text
¿Qué parte de la request fue lenta?
¿Qué dependencia falló?
¿Qué endpoint tiene errores?
¿Cuánto tarda DB/API externa?
```

Patrón:

```text
Usuario -> ALB -> App EC2 -> RDS / SQS / Lambda
```

Buenas prácticas:

```text
Propaga trace IDs.
Instrumenta framework web.
Instrumenta llamadas HTTP y DB si aplica.
No captures datos sensibles.
Usa sampling para controlar volumen.
Combina X-Ray con CloudWatch.
```

---

## 45. Alarmas con SNS

Patrón:

```text
CloudWatch Alarm -> SNS Topic -> Email / Chatbot / Lambda / Pager
```

Buenas prácticas:

```text
Define severidades.
Evita alertas ruidosas.
Documenta runbook.
Incluye link al dashboard.
Prueba alertas.
```

---

# Parte IX: infraestructura como código con CloudFormation

## 46. Por qué CloudFormation

CloudFormation permite definir recursos AWS como plantillas.

Ventajas:

```text
Infraestructura versionada.
Reproducibilidad.
Revisión por pull request.
Rollback.
Stacks por ambiente.
Menos cambios manuales.
```

Conceptos:

```text
Template:
Archivo YAML/JSON con recursos.

Stack:
Instancia desplegada de un template.

Change set:
Vista previa de cambios antes de aplicarlos.

Outputs:
Valores exportables o visibles.

Parameters:
Valores configurables por ambiente.
```

---

## 47. Estructura de stacks

Recomendación:

```text
network-stack:
VPC, subredes, route tables, gateways, endpoints.

security-stack:
IAM roles, security groups, KMS.

data-stack:
RDS, S3 buckets, Secrets.

app-stack:
ALB, target groups, launch template, ASG.

observability-stack:
CloudWatch alarms, dashboards, CloudTrail, SNS topics.
```

Ventajas:

```text
Separación por ciclo de vida.
Menos riesgo al actualizar.
Reutilización entre ambientes.
```

---

## 48. CloudFormation: VPC mínima

Ejemplo conceptual simplificado:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: VPC base para aplicacion web

Parameters:
  VpcCidr:
    Type: String
    Default: 10.0.0.0/16

Resources:
  WebVpc:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCidr
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: web-prod-vpc

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: web-prod-igw

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref WebVpc
      InternetGatewayId: !Ref InternetGateway
```

---

## 49. CloudFormation: subred pública

```yaml
PublicSubnetA:
  Type: AWS::EC2::Subnet
  Properties:
    VpcId: !Ref WebVpc
    CidrBlock: 10.0.1.0/24
    AvailabilityZone: !Select [0, !GetAZs ""]
    MapPublicIpOnLaunch: true
    Tags:
      - Key: Name
        Value: web-prod-public-a

PublicRouteTable:
  Type: AWS::EC2::RouteTable
  Properties:
    VpcId: !Ref WebVpc

PublicDefaultRoute:
  Type: AWS::EC2::Route
  DependsOn: AttachGateway
  Properties:
    RouteTableId: !Ref PublicRouteTable
    DestinationCidrBlock: 0.0.0.0/0
    GatewayId: !Ref InternetGateway

PublicSubnetARouteTableAssociation:
  Type: AWS::EC2::SubnetRouteTableAssociation
  Properties:
    SubnetId: !Ref PublicSubnetA
    RouteTableId: !Ref PublicRouteTable
```

---

## 50. CloudFormation: subred privada y NAT

```yaml
NatEipA:
  Type: AWS::EC2::EIP
  Properties:
    Domain: vpc

NatGatewayA:
  Type: AWS::EC2::NatGateway
  Properties:
    AllocationId: !GetAtt NatEipA.AllocationId
    SubnetId: !Ref PublicSubnetA

PrivateSubnetA:
  Type: AWS::EC2::Subnet
  Properties:
    VpcId: !Ref WebVpc
    CidrBlock: 10.0.11.0/24
    AvailabilityZone: !Select [0, !GetAZs ""]
    MapPublicIpOnLaunch: false
    Tags:
      - Key: Name
        Value: web-prod-private-app-a

PrivateRouteTableA:
  Type: AWS::EC2::RouteTable
  Properties:
    VpcId: !Ref WebVpc

PrivateDefaultRouteA:
  Type: AWS::EC2::Route
  Properties:
    RouteTableId: !Ref PrivateRouteTableA
    DestinationCidrBlock: 0.0.0.0/0
    NatGatewayId: !Ref NatGatewayA
```

---

## 51. CloudFormation: security groups

```yaml
AlbSecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: Public ALB security group
    VpcId: !Ref WebVpc
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 443
        ToPort: 443
        CidrIp: 0.0.0.0/0
    SecurityGroupEgress:
      - IpProtocol: -1
        CidrIp: 0.0.0.0/0

AppSecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: Private app security group
    VpcId: !Ref WebVpc
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 8000
        ToPort: 8000
        SourceSecurityGroupId: !Ref AlbSecurityGroup
```

---

## 52. CloudFormation: ALB básico

```yaml
ApplicationLoadBalancer:
  Type: AWS::ElasticLoadBalancingV2::LoadBalancer
  Properties:
    Scheme: internet-facing
    Type: application
    SecurityGroups:
      - !Ref AlbSecurityGroup
    Subnets:
      - !Ref PublicSubnetA
      - !Ref PublicSubnetB

AppTargetGroup:
  Type: AWS::ElasticLoadBalancingV2::TargetGroup
  Properties:
    VpcId: !Ref WebVpc
    Protocol: HTTP
    Port: 8000
    TargetType: instance
    HealthCheckPath: /health
    Matcher:
      HttpCode: "200"

HttpsListener:
  Type: AWS::ElasticLoadBalancingV2::Listener
  Properties:
    LoadBalancerArn: !Ref ApplicationLoadBalancer
    Port: 443
    Protocol: HTTPS
    Certificates:
      - CertificateArn: !Ref CertificateArn
    DefaultActions:
      - Type: forward
        TargetGroupArn: !Ref AppTargetGroup
```

---

## 53. Despliegue de CloudFormation

Crear stack:

```bash
aws cloudformation deploy \
    --template-file network.yaml \
    --stack-name web-prod-network \
    --capabilities CAPABILITY_NAMED_IAM \
    --parameter-overrides Environment=prod
```

Ver cambios antes:

```bash
aws cloudformation create-change-set \
    --stack-name web-prod-network \
    --template-body file://network.yaml \
    --change-set-name preview-change
```

Buenas prácticas:

```text
Usa change sets para producción.
Separa parámetros por ambiente.
No guardes secretos en templates.
Usa Outputs y exports con criterio.
Protege stacks críticos contra eliminación accidental.
```

---

# Parte X: operación productiva

## 54. Checklist de red

```text
[ ] VPC no default.
[ ] Al menos 2 AZ.
[ ] Subredes públicas para ALB/NAT.
[ ] Subredes privadas para app.
[ ] Subredes privadas para datos.
[ ] IGW asociado.
[ ] NAT Gateway configurado.
[ ] Route tables correctas.
[ ] RDS sin acceso público.
[ ] VPC endpoints para S3 y servicios críticos si aplica.
```

---

## 55. Checklist de seguridad

```text
[ ] Root account protegido con MFA.
[ ] IAM Identity Center para humanos.
[ ] Roles para workloads.
[ ] Mínimo privilegio.
[ ] Security groups por función.
[ ] Sin SSH público.
[ ] RDS privado.
[ ] S3 Block Public Access.
[ ] Secrets en Secrets Manager.
[ ] Configuración en Parameter Store.
[ ] CloudTrail activo.
[ ] Logs sin secretos.
```

---

## 56. Checklist de aplicación web

```text
[ ] ALB con HTTPS.
[ ] Certificado ACM válido.
[ ] HTTP redirige a HTTPS.
[ ] Health check /health.
[ ] App stateless.
[ ] Auto Scaling Group en subredes privadas.
[ ] Sesiones fuera de EC2.
[ ] Deploy reproducible.
[ ] Rollback definido.
[ ] Variables por ambiente.
```

---

## 57. Checklist de base de datos

```text
[ ] RDS Multi-AZ.
[ ] Backups automáticos.
[ ] Retention definida.
[ ] Cifrado en reposo.
[ ] Security group solo desde app.
[ ] Usuario app sin privilegios excesivos.
[ ] Migraciones versionadas.
[ ] Restauración probada.
[ ] Alarmas de storage/conexiones/CPU.
```

---

## 58. Checklist de observabilidad

```text
[ ] CloudWatch Logs configurado.
[ ] Retention definida.
[ ] Métricas de ALB, EC2, RDS, SQS y Lambda.
[ ] Alarmas críticas.
[ ] SNS topic para alertas.
[ ] Dashboard productivo.
[ ] CloudTrail multi-region.
[ ] X-Ray en endpoints críticos.
[ ] Correlation ID en logs.
```

---

## 59. Checklist de costos

```text
[ ] NAT Gateway revisado.
[ ] Logs con retención definida.
[ ] CloudFront cache configurado.
[ ] RDS dimensionado correctamente.
[ ] EC2 con Auto Scaling.
[ ] Savings Plans/Reserved Instances evaluados.
[ ] S3 lifecycle policies.
[ ] Alarmas de presupuesto.
[ ] Recursos huérfanos revisados.
```

Servicios que suelen sorprender en costos:

```text
NAT Gateway por datos procesados.
CloudWatch Logs sin retención.
RDS sobredimensionado.
Snapshots olvidados.
Data transfer cross-AZ.
CloudFront invalidations excesivas.
```

---

## 60. Runbook mínimo de incidente

Ejemplo: aumento de errores 5xx.

```text
1. Revisar CloudWatch Alarm.
2. Abrir dashboard de ALB.
3. Ver si son ELB 5xx o Target 5xx.
4. Revisar HealthyHostCount.
5. Revisar logs de app por request_id.
6. Revisar despliegue reciente.
7. Revisar RDS CPU/conexiones/storage.
8. Revisar SQS/Lambda si el flujo es asíncrono.
9. Si empezó tras deploy, ejecutar rollback.
10. Documentar causa y acciones.
```

---

# Parte XI: ejemplo de flujo completo

## 61. Flujo de request web

```text
1. Usuario entra a https://app.example.com.
2. Route 53 resuelve dominio.
3. CloudFront sirve frontend desde S3 privado.
4. Frontend llama /api/tasks.
5. CloudFront enruta /api/* al ALB.
6. ALB envía request a EC2 privada saludable.
7. App valida JWT de Cognito.
8. App consulta RDS privado.
9. App guarda logs en CloudWatch.
10. App envía tarea pesada a SQS si aplica.
11. Worker/Lambda procesa SQS.
12. X-Ray muestra traza de la request.
13. CloudTrail registra cambios administrativos.
```

---

## 62. Flujo de upload

```text
1. Usuario pide subir archivo.
2. Backend valida autenticación y permisos.
3. Backend genera presigned URL para S3.
4. Frontend sube archivo directamente a S3.
5. S3 emite evento.
6. EventBridge o S3 Notification activa Lambda.
7. Lambda procesa archivo.
8. Metadata se guarda en RDS.
9. Resultado se notifica vía SNS/SQS o queda disponible para consulta.
```

---

## 63. Flujo asíncrono con SQS

```text
1. Usuario crea reporte.
2. API guarda solicitud en RDS.
3. API envía mensaje a SQS.
4. Worker toma mensaje.
5. Worker genera reporte.
6. Worker guarda archivo en S3.
7. Worker actualiza estado en RDS.
8. Usuario consulta estado.
9. Si falla repetidamente, mensaje va a DLQ.
```

---

# Parte XII: resumen de reglas principales

```text
1. Usa VPC propia para producción.
2. Usa mínimo dos AZ.
3. Coloca ALB y NAT en subredes públicas.
4. Coloca EC2 backend en subredes privadas.
5. Coloca RDS en subredes privadas de datos.
6. No abras RDS a internet.
7. Usa Security Groups referenciados entre sí.
8. Usa NAT Gateway para salida desde privadas y VPC endpoints cuando convenga.
9. Usa ALB con HTTPS y health checks.
10. Usa Auto Scaling Group para alta disponibilidad de EC2.
11. Usa S3 privado + CloudFront OAC para frontend estático.
12. Usa Cognito para autenticación cuando quieras identidad gestionada.
13. Usa IAM roles, no access keys en instancias.
14. Usa Secrets Manager para secretos y Parameter Store para configuración.
15. Usa SQS para trabajos asíncronos y DLQ para fallos.
16. Usa SNS para fan-out y alertas.
17. Usa EventBridge para eventos de dominio y scheduling.
18. Usa CloudFormation para infraestructura versionada.
19. Usa CloudWatch para logs, métricas, alarmas y dashboards.
20. Usa CloudTrail para auditoría de cuenta.
21. Usa X-Ray para trazas distribuidas.
22. Diseña con rollback, backups y monitoreo desde el inicio.
```

---

# Fuentes de referencia recomendadas

```text
- AWS VPC User Guide.
- AWS EC2 User Guide.
- Elastic Load Balancing User Guide.
- Amazon EC2 Auto Scaling User Guide.
- Amazon RDS User Guide.
- Amazon S3 User Guide.
- Amazon CloudFront Developer Guide.
- Amazon Cognito Developer Guide.
- AWS IAM User Guide.
- AWS Secrets Manager User Guide.
- AWS Systems Manager Parameter Store User Guide.
- Amazon SQS Developer Guide.
- Amazon SNS Developer Guide.
- Amazon EventBridge User Guide.
- AWS Lambda Developer Guide.
- AWS CloudFormation User Guide.
- Amazon CloudWatch User Guide.
- AWS CloudTrail User Guide.
- AWS X-Ray Developer Guide.
```
