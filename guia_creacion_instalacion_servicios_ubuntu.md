# Guía de creación, instalación y operación de servicios en Ubuntu

## Objetivo

Esta guía explica cómo crear, instalar, configurar, gobernar, monitorear y operar servicios en Ubuntu usando `systemd`. Está orientada a aplicaciones propias: APIs Python, workers, scripts recurrentes, procesos de background, jobs, servicios internos, demonios y utilidades de operación.

Está organizada en tres niveles:

```text
1. Básico: conceptos, creación de un servicio, systemctl, journalctl, logs y ciclo de vida.
2. Intermedio: configuración por ambiente, recursos, usuarios, permisos, timers, watchdogs, restart policies y monitoreo.
3. Avanzado: hardening, gobernanza, plantillas, slices, cgroups, despliegue, auditoría, runbooks y troubleshooting.
```

La guía usa Ubuntu moderno, donde `systemd` es el sistema de inicialización y gestión de servicios.

---

# Parte I: fundamentos

## 1. Qué es un servicio en Ubuntu

Un servicio es un proceso administrado por el sistema operativo. Normalmente debe:

```text
Arrancar automáticamente.
Reiniciarse si falla.
Escribir logs.
Ejecutarse con un usuario específico.
Tener límites de recursos.
Exponer estado.
Detenerse limpiamente.
Integrarse con monitoreo.
```

Ejemplos:

```text
API FastAPI con Uvicorn.
Worker que consume SQS/RabbitMQ/Redis.
Script de sincronización.
Job de ETL.
Servidor interno.
Proceso de notificaciones.
Bot.
Scheduler.
```

En Ubuntu, la herramienta estándar para gestionar servicios es `systemd`.

---

## 2. Qué es systemd

`systemd` es el administrador de sistema y servicios. Se ejecuta como PID 1 y se encarga de arrancar, detener, reiniciar, supervisar y coordinar procesos del sistema.

Herramientas principales:

```text
systemctl:
Gestiona servicios y unidades.

journalctl:
Consulta logs del journal.

systemd-analyze:
Analiza arranque, dependencias y tiempos.

loginctl:
Gestiona sesiones de usuarios.

timedatectl:
Gestiona fecha y zona horaria.

hostnamectl:
Gestiona nombre del host.
```

---

## 3. Unit files

`systemd` usa archivos llamados unit files. Un unit file describe un recurso del sistema.

Tipos comunes:

```text
.service    servicio o proceso.
.timer      programación temporal, alternativa a cron.
.socket     activación por socket.
.target     agrupación de unidades.
.mount      punto de montaje.
.path       activación por cambios en archivos.
.slice      agrupación de recursos con cgroups.
```

Esta guía se centra en:

```text
.service
.timer
.slice
```

---

## 4. Ubicación de archivos de servicio

Rutas principales:

```text
/etc/systemd/system/
    Servicios creados o modificados por el administrador.
    Aquí debes poner servicios propios.

/lib/systemd/system/
    Servicios instalados por paquetes del sistema.
    No edites aquí directamente.

/run/systemd/system/
    Unidades temporales de runtime.
```

Regla:

```text
Para servicios propios, usa /etc/systemd/system/.
```

Ejemplo:

```bash
sudo nano /etc/systemd/system/myapp.service
```

---

## 5. Anatomía de un servicio

Ejemplo mínimo:

```ini
[Unit]
Description=My App Service
After=network.target

[Service]
Type=simple
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/.venv/bin/python -m myapp
Restart=on-failure
User=myapp
Group=myapp

[Install]
WantedBy=multi-user.target
```

Secciones:

```text
[Unit]:
Metadata y dependencias.

[Service]:
Cómo se ejecuta el proceso.

[Install]:
Cómo se habilita al arranque.
```

---

# Parte II: primer servicio

## 6. Crear usuario de servicio

No ejecutes aplicaciones propias como `root` salvo necesidad clara.

Crear usuario de sistema:

```bash
sudo useradd --system --create-home --shell /usr/sbin/nologin myapp
```

Ver:

```bash
id myapp
```

Ventajas:

```text
Menor impacto si el proceso se compromete.
Permisos más claros.
Mejor auditoría.
Separación entre servicios.
```

---

## 7. Crear directorio de aplicación

```bash
sudo mkdir -p /opt/myapp
sudo chown -R myapp:myapp /opt/myapp
```

Para logs propios si decides escribir archivos además del journal:

```bash
sudo mkdir -p /var/log/myapp
sudo chown myapp:myapp /var/log/myapp
```

Para datos persistentes:

```bash
sudo mkdir -p /var/lib/myapp
sudo chown myapp:myapp /var/lib/myapp
```

Para configuración:

```bash
sudo mkdir -p /etc/myapp
sudo chown root:root /etc/myapp
sudo chmod 755 /etc/myapp
```

Convención recomendada:

```text
/opt/myapp        código o artefacto desplegado.
/etc/myapp        configuración.
/var/lib/myapp    datos persistentes.
/var/log/myapp    logs en archivo, si aplica.
/run/myapp        archivos temporales de runtime.
```

---

## 8. Crear aplicación Python mínima

Crear archivo:

```bash
sudo -u myapp nano /opt/myapp/app.py
```

Contenido:

```python
import logging
import time

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(message)s",
)

logger = logging.getLogger("myapp")

while True:
    logger.info("service is alive")
    time.sleep(10)
```

Probar manualmente:

```bash
sudo -u myapp python3 /opt/myapp/app.py
```

Detener con:

```text
Ctrl+C
```

---

## 9. Crear unit file

```bash
sudo nano /etc/systemd/system/myapp.service
```

Contenido:

```ini
[Unit]
Description=My App background service
Documentation=https://example.com/docs/myapp
After=network.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Guardar y recargar systemd:

```bash
sudo systemctl daemon-reload
```

Iniciar:

```bash
sudo systemctl start myapp
```

Ver estado:

```bash
sudo systemctl status myapp
```

Habilitar arranque automático:

```bash
sudo systemctl enable myapp
```

---

## 10. Ciclo de vida con systemctl

Comandos básicos:

```bash
sudo systemctl start myapp
sudo systemctl stop myapp
sudo systemctl restart myapp
sudo systemctl reload myapp
sudo systemctl status myapp
sudo systemctl enable myapp
sudo systemctl disable myapp
```

Ver si está activo:

```bash
systemctl is-active myapp
```

Ver si está habilitado al boot:

```bash
systemctl is-enabled myapp
```

Ver unidades fallidas:

```bash
systemctl --failed
```

Resetear estado fallido:

```bash
sudo systemctl reset-failed myapp
```

---

## 11. `daemon-reload`

Cada vez que creas o modificas un unit file:

```bash
sudo systemctl daemon-reload
```

Después:

```bash
sudo systemctl restart myapp
```

Regla:

```text
Editar unit file sin daemon-reload puede hacer que systemd siga usando la versión anterior.
```

---

## 12. Ver configuración efectiva

Mostrar unit file:

```bash
systemctl cat myapp
```

Mostrar propiedades:

```bash
systemctl show myapp
```

Ver propiedad específica:

```bash
systemctl show myapp -p ExecStart
systemctl show myapp -p MainPID
systemctl show myapp -p MemoryCurrent
systemctl show myapp -p CPUUsageNSec
```

---

# Parte III: logs y consola

## 13. Logs con journalctl

`journalctl` consulta el journal de systemd.

Ver logs de un servicio:

```bash
journalctl -u myapp
```

Seguir en vivo:

```bash
journalctl -u myapp -f
```

Últimas 100 líneas:

```bash
journalctl -u myapp -n 100
```

Desde el último boot:

```bash
journalctl -u myapp -b
```

Con timestamps ISO:

```bash
journalctl -u myapp -o short-iso
```

Filtrar por tiempo:

```bash
journalctl -u myapp --since "1 hour ago"
journalctl -u myapp --since "2026-07-04 10:00:00"
journalctl -u myapp --until "2026-07-04 11:00:00"
```

Solo errores:

```bash
journalctl -u myapp -p err
```

---

## 14. Logs de consola

Si tu proceso escribe en stdout o stderr, systemd lo captura por defecto.

Python:

```python
print("hello from stdout")
```

Logging:

```python
import logging

logging.basicConfig(level=logging.INFO)
logging.info("service started")
```

Ver:

```bash
journalctl -u myapp -f
```

Regla:

```text
Para servicios modernos, escribir logs a stdout/stderr suele ser suficiente.
systemd-journald los captura y los hace consultables con journalctl.
```

---

## 15. Configurar salida estándar y error

En `[Service]`:

```ini
StandardOutput=journal
StandardError=journal
```

Esto suele ser el comportamiento por defecto, pero declararlo puede aclarar intención.

Para enviar a archivo:

```ini
StandardOutput=append:/var/log/myapp/output.log
StandardError=append:/var/log/myapp/error.log
```

Advertencias:

```text
Si escribes a archivos, debes gestionar rotación.
El journal ya ofrece retención y consulta estructurada.
No dupliques logs sin necesidad.
```

---

## 16. Logs estructurados

Para servicios productivos, usa logs estructurados.

Ejemplo Python:

```python
import json
import logging
import sys

logger = logging.getLogger("myapp")
logger.setLevel(logging.INFO)

handler = logging.StreamHandler(sys.stdout)
logger.addHandler(handler)

def log_event(event: str, **fields: object) -> None:
    logger.info(json.dumps({
        "event": event,
        **fields,
    }))

log_event("service_started", version="1.0.0")
```

Ver:

```bash
journalctl -u myapp -o cat
```

Buenas prácticas:

```text
Incluye service, environment, version.
Incluye request_id/correlation_id si aplica.
No loguees secretos.
No loguees tokens.
No loguees datos personales innecesarios.
```

---

## 17. Persistencia del journal

En muchos sistemas, journald puede guardar logs de forma persistente si existe:

```bash
/var/log/journal
```

Crear:

```bash
sudo mkdir -p /var/log/journal
sudo systemctl restart systemd-journald
```

Configurar:

```bash
sudo nano /etc/systemd/journald.conf
```

Opciones comunes:

```ini
[Journal]
Storage=persistent
SystemMaxUse=1G
MaxRetentionSec=30day
```

Reiniciar journald:

```bash
sudo systemctl restart systemd-journald
```

Ver uso:

```bash
journalctl --disk-usage
```

Limpiar logs antiguos:

```bash
sudo journalctl --vacuum-time=30d
sudo journalctl --vacuum-size=1G
```

---

# Parte IV: servicios Python reales

## 18. Servicio Python con entorno virtual

Crear app:

```bash
sudo mkdir -p /opt/myapi
sudo chown -R myapp:myapp /opt/myapi
```

Crear venv:

```bash
sudo -u myapp python3 -m venv /opt/myapi/.venv
```

Instalar dependencias:

```bash
sudo -u myapp /opt/myapi/.venv/bin/pip install fastapi uvicorn
```

Ejemplo app:

```bash
sudo -u myapp nano /opt/myapi/main.py
```

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/health")
def health() -> dict[str, str]:
    return {"status": "ok"}
```

Servicio:

```ini
[Unit]
Description=My FastAPI service
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapi
Environment=APP_ENV=production
ExecStart=/opt/myapi/.venv/bin/uvicorn main:app --host 127.0.0.1 --port 8000
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Instalar:

```bash
sudo nano /etc/systemd/system/myapi.service
sudo systemctl daemon-reload
sudo systemctl enable --now myapi
```

Probar:

```bash
curl http://127.0.0.1:8000/health
```

---

## 19. Variables de entorno

Puedes usar `Environment=`:

```ini
[Service]
Environment=APP_ENV=production
Environment=LOG_LEVEL=INFO
```

Para varias variables, usa `EnvironmentFile=`.

Archivo:

```bash
sudo nano /etc/myapi/myapi.env
```

Contenido:

```env
APP_ENV=production
LOG_LEVEL=INFO
DATABASE_URL=postgresql://app_user:password@localhost:5432/app_db
```

Permisos:

```bash
sudo chown root:myapp /etc/myapi/myapi.env
sudo chmod 640 /etc/myapi/myapi.env
```

Servicio:

```ini
[Service]
EnvironmentFile=/etc/myapi/myapi.env
```

Advertencia:

```text
EnvironmentFile no es un gestor de secretos.
Para secretos sensibles, considera un secret manager, archivo protegido, systemd credentials o una integración externa.
```

---

## 20. Servicio con reload

Algunos servicios soportan recarga de configuración sin reiniciar.

Ejemplo:

```ini
[Service]
ExecReload=/bin/kill -HUP $MAINPID
```

Usar:

```bash
sudo systemctl reload myapp
```

Si tu aplicación no maneja SIGHUP, no agregues reload falso.

Regla:

```text
reload debe recargar configuración sin cortar el servicio.
Si no existe esa capacidad, usa restart.
```

---

## 21. Señales de parada

Por defecto systemd envía SIGTERM y luego SIGKILL si el servicio no se detiene.

Configurar tiempo:

```ini
[Service]
TimeoutStopSec=30
KillSignal=SIGTERM
```

En Python, maneja SIGTERM si necesitas cierre limpio:

```python
import signal
import sys
import time

running = True

def handle_sigterm(signum, frame):
    global running
    running = False

signal.signal(signal.SIGTERM, handle_sigterm)

while running:
    time.sleep(1)

sys.exit(0)
```

Buenas prácticas:

```text
Cerrar conexiones.
Terminar requests en curso si aplica.
Confirmar mensajes procesados.
Liberar locks.
Escribir logs de shutdown.
```

---

# Parte V: configuración de recursos

## 22. Por qué limitar recursos

Sin límites, un proceso defectuoso puede consumir:

```text
Toda la memoria.
CPU excesiva.
Demasiados archivos.
Demasiados procesos.
Demasiado I/O.
```

`systemd` permite aplicar límites usando cgroups.

---

## 23. Limitar memoria

En `[Service]`:

```ini
MemoryMax=512M
```

Ejemplo:

```ini
[Service]
MemoryMax=512M
Restart=on-failure
```

También puedes usar:

```ini
MemoryHigh=400M
MemoryMax=512M
```

Idea:

```text
MemoryHigh:
Límite suave que presiona al servicio.

MemoryMax:
Límite duro.
```

Ver memoria actual:

```bash
systemctl show myapp -p MemoryCurrent
```

O:

```bash
systemctl status myapp
```

---

## 24. Limitar CPU

```ini
CPUQuota=50%
```

Significado aproximado:

```text
50% de un core.
100% de un core.
200% de dos cores.
```

Ejemplo:

```ini
[Service]
CPUQuota=150%
```

Ver uso:

```bash
systemctl show myapp -p CPUUsageNSec
```

---

## 25. Limitar cantidad de procesos

```ini
TasksMax=100
```

Útil para evitar forks excesivos.

Ver:

```bash
systemctl show myapp -p TasksCurrent -p TasksMax
```

---

## 26. Limitar archivos abiertos

```ini
LimitNOFILE=65535
```

Útil en servicios con muchas conexiones.

Ver límites de un proceso:

```bash
cat /proc/$(systemctl show -p MainPID --value myapp)/limits
```

---

## 27. Limitar I/O

Ejemplos:

```ini
IOWeight=100
```

O restricciones específicas, según soporte del sistema:

```ini
IOReadBandwidthMax=/dev/nvme0n1 10M
IOWriteBandwidthMax=/dev/nvme0n1 10M
```

Regla:

```text
Prueba límites de I/O en staging.
No todos los entornos soportan todas las opciones igual.
```

---

## 28. Aplicar recursos sin editar archivo

Puedes usar:

```bash
sudo systemctl set-property myapp.service MemoryMax=512M CPUQuota=50%
```

Esto crea overrides persistentes.

Ver override:

```bash
systemctl cat myapp
```

Quitar o editar:

```bash
sudo systemctl edit myapp
```

---

# Parte VI: overrides y configuración segura

## 29. `systemctl edit`

No edites unit files de paquetes en `/lib/systemd/system`. Usa overrides.

Crear override:

```bash
sudo systemctl edit myapp
```

Esto abre un archivo en:

```text
/etc/systemd/system/myapp.service.d/override.conf
```

Ejemplo:

```ini
[Service]
Environment=LOG_LEVEL=DEBUG
MemoryMax=1G
```

Aplicar:

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp
```

Ver configuración final:

```bash
systemctl cat myapp
```

---

## 30. Drop-in files

Puedes crear manualmente:

```bash
sudo mkdir -p /etc/systemd/system/myapp.service.d
sudo nano /etc/systemd/system/myapp.service.d/resources.conf
```

Contenido:

```ini
[Service]
MemoryMax=512M
CPUQuota=100%
```

Aplicar:

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp
```

Ventaja:

```text
Permite separar configuración base, recursos, seguridad y ambiente.
```

---

## 31. Validar unit files

Verificar sintaxis y problemas comunes:

```bash
systemd-analyze verify /etc/systemd/system/myapp.service
```

Recargar:

```bash
sudo systemctl daemon-reload
```

---

# Parte VII: reinicio, watchdog y salud

## 32. Restart policies

Opciones frecuentes:

```ini
Restart=no
Restart=on-success
Restart=on-failure
Restart=on-abnormal
Restart=on-abort
Restart=always
```

Recomendación común para servicios de larga duración:

```ini
Restart=on-failure
RestartSec=5
```

Para workers críticos:

```ini
Restart=always
RestartSec=10
```

Evita loops agresivos:

```ini
StartLimitIntervalSec=300
StartLimitBurst=5
```

Ejemplo:

```ini
[Unit]
StartLimitIntervalSec=300
StartLimitBurst=5

[Service]
Restart=on-failure
RestartSec=10
```

Significado:

```text
Máximo 5 intentos de inicio en 300 segundos.
```

---

## 33. Health check externo

Para una API local:

```bash
curl -f http://127.0.0.1:8000/health
```

Puedes usar monitoreo externo:

```text
Prometheus.
Nagios/Icinga.
Zabbix.
Datadog.
New Relic.
CloudWatch Agent.
Uptime Kuma.
Grafana Agent.
```

systemd no reemplaza monitoreo de aplicación.

---

## 34. Watchdog con systemd

`WatchdogSec` permite que systemd reinicie un servicio si deja de notificar que está vivo.

Unit:

```ini
[Service]
Type=notify
WatchdogSec=30
Restart=on-failure
```

La app debe enviar notificaciones a systemd. En Python puedes usar librerías como `sdnotify`.

Ejemplo conceptual:

```python
import time
from sdnotify import SystemdNotifier

notifier = SystemdNotifier()
notifier.notify("READY=1")

while True:
    notifier.notify("WATCHDOG=1")
    time.sleep(10)
```

Regla:

```text
No configures Type=notify o WatchdogSec si la app no implementa el protocolo de notificación.
```

---

## 35. Readiness vs liveness

Distinción útil:

```text
Liveness:
El proceso está vivo.

Readiness:
El servicio está listo para recibir tráfico real.
```

Ejemplos:

```text
Liveness:
El proceso Python sigue ejecutándose.

Readiness:
La API responde /health y puede conectarse a la base de datos.
```

En systemd puedes saber si el proceso vive, pero la readiness normalmente debe ser verificada por:

```text
Health endpoint.
Load balancer.
Script de monitoreo.
Orquestador.
```

---

# Parte VIII: timers

## 36. Cuándo usar timers

`systemd timers` son alternativa a cron.

Usos:

```text
Jobs recurrentes.
Backups.
Limpieza.
Sincronizaciones.
Reportes.
Mantenimiento.
```

Ventajas frente a cron:

```text
Integración con journalctl.
Dependencias.
Control con systemctl.
Estado visible.
Missed runs con Persistent=true.
```

---

## 37. Crear servicio oneshot

Archivo:

```bash
sudo nano /etc/systemd/system/mybackup.service
```

Contenido:

```ini
[Unit]
Description=Run my backup job

[Service]
Type=oneshot
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/.venv/bin/python /opt/myapp/scripts/backup.py
```

---

## 38. Crear timer

Archivo:

```bash
sudo nano /etc/systemd/system/mybackup.timer
```

Contenido:

```ini
[Unit]
Description=Run my backup job daily

[Timer]
OnCalendar=*-*-* 02:30:00
Persistent=true
Unit=mybackup.service

[Install]
WantedBy=timers.target
```

Activar:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now mybackup.timer
```

Ver timers:

```bash
systemctl list-timers
```

Ver logs:

```bash
journalctl -u mybackup.service
```

Ejecutar manualmente:

```bash
sudo systemctl start mybackup.service
```

---

## 39. Ejemplos de OnCalendar

Diario a las 02:30:

```ini
OnCalendar=*-*-* 02:30:00
```

Cada hora:

```ini
OnCalendar=hourly
```

Lunes a las 03:00:

```ini
OnCalendar=Mon *-*-* 03:00:00
```

Cada 15 minutos:

```ini
OnCalendar=*:0/15
```

Después de boot:

```ini
OnBootSec=5min
OnUnitActiveSec=1h
```

---

# Parte IX: dependencias y orden de arranque

## 40. After, Wants y Requires

Conceptos:

```text
After:
Orden de arranque. No implica dependencia fuerte.

Wants:
Dependencia débil. Intenta iniciar otra unidad, pero no falla si esa unidad falla.

Requires:
Dependencia fuerte. Si la dependencia falla, esta unidad puede fallar.
```

Ejemplo:

```ini
[Unit]
After=network-online.target
Wants=network-online.target
```

Uso típico para servicios de red:

```ini
After=network-online.target
Wants=network-online.target
```

---

## 41. Servicio que depende de PostgreSQL

Si PostgreSQL está en la misma máquina:

```ini
[Unit]
Description=My API
After=network-online.target postgresql.service
Wants=network-online.target
Requires=postgresql.service
```

Cuidado:

```text
Si la base de datos es remota, no uses Requires=postgresql.service.
En ese caso usa retry/backoff en la aplicación y health checks.
```

---

## 42. ExecStartPre

Ejecuta comandos antes del servicio.

Ejemplo:

```ini
[Service]
ExecStartPre=/usr/bin/test -f /etc/myapp/myapp.env
ExecStart=/opt/myapp/.venv/bin/python -m myapp
```

Para migraciones:

```ini
ExecStartPre=/opt/myapp/.venv/bin/python -m myapp.migrate
```

Advertencia:

```text
Ejecutar migraciones automáticas antes de cada arranque puede ser riesgoso en producción si hay varias instancias.
```

---

## 43. ExecStopPost

Acción posterior al stop:

```ini
[Service]
ExecStopPost=/usr/bin/logger "myapp stopped"
```

Útil para:

```text
Limpieza.
Auditoría.
Notificación local.
```

No debe ser lógica crítica compleja.

---

# Parte X: seguridad y hardening

## 44. Principios

Un servicio productivo debe:

```text
No correr como root.
Tener permisos mínimos.
Tener límites de recursos.
Tener filesystem restringido si aplica.
No poder escribir donde no necesita.
No ver secretos ajenos.
No acceder a red si no necesita.
Registrar logs.
Tener configuración versionada.
```

---

## 45. Opciones de hardening útiles

En `[Service]`:

```ini
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/myapp /var/log/myapp
```

Significado:

```text
NoNewPrivileges:
Evita ganar privilegios nuevos.

PrivateTmp:
Da /tmp privado al servicio.

ProtectSystem=strict:
Monta partes del sistema como solo lectura.

ProtectHome=true:
Bloquea acceso a /home, /root y /run/user.

ReadWritePaths:
Permite escritura solo en rutas específicas.
```

Ejemplo:

```ini
[Service]
User=myapp
Group=myapp
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/myapp /var/log/myapp
```

---

## 46. Restringir capacidades

Si el servicio no necesita capacidades especiales:

```ini
CapabilityBoundingSet=
AmbientCapabilities=
```

Esto elimina capacidades Linux adicionales.

Si necesita bind a puerto bajo, como 80, puedes usar:

```ini
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
```

Pero en general es mejor:

```text
Apache/Nginx escucha 80/443.
Tu app escucha 127.0.0.1:8000 sin privilegios.
```

---

## 47. Restringir red

Si un servicio no necesita red:

```ini
PrivateNetwork=true
```

Si necesita red, no uses esa opción.

Puedes restringir familias de direcciones:

```ini
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX
```

---

## 48. System call filtering

Ejemplo:

```ini
SystemCallArchitectures=native
RestrictRealtime=true
LockPersonality=true
MemoryDenyWriteExecute=true
```

Más estricto:

```ini
SystemCallFilter=@system-service
```

Advertencia:

```text
Estas opciones pueden romper aplicaciones.
Actívalas progresivamente y prueba en staging.
```

---

## 49. `systemd-analyze security`

Evaluar exposición de seguridad:

```bash
systemd-analyze security myapp.service
```

Entrega una evaluación orientativa de hardening.

Regla:

```text
No busques una puntuación perfecta sin entender impacto.
Usa el reporte como guía para endurecer gradualmente.
```

---

# Parte XI: gobernanza

## 50. Qué significa gobernanza de servicios

Gobernanza significa que los servicios están documentados, controlados y operados bajo reglas claras.

Incluye:

```text
Propietario.
Propósito.
Ambiente.
Dependencias.
Permisos.
Recursos.
Logs.
Monitoreo.
Runbook.
SLO/SLA.
Procedimiento de deploy.
Procedimiento de rollback.
Retención de datos.
Clasificación de criticidad.
```

---

## 51. Estándar de nombres

Usa nombres consistentes.

Ejemplos:

```text
myapp-api.service
myapp-worker.service
myapp-scheduler.service
myapp-backup.timer
myapp-backup.service
```

Evita:

```text
test.service
app.service
new.service
python.service
```

Formato recomendado:

```text
<producto>-<componente>.service
```

Ejemplo:

```text
billing-api.service
billing-worker.service
billing-reports.timer
```

---

## 52. Etiquetado documental

systemd no tiene tags como tal, pero puedes documentar en `[Unit]`.

```ini
[Unit]
Description=Billing API service
Documentation=https://wiki.example.com/billing-api
```

También puedes mantener un inventario externo.

Ejemplo de inventario:

```yaml
service: billing-api.service
owner: platform-team
environment: production
criticality: high
language: python
runtime: uvicorn
port: 8000
user: billing
data_paths:
  - /var/lib/billing
logs:
  - journalctl -u billing-api
dependencies:
  - postgresql
  - redis
runbook: https://wiki.example.com/runbooks/billing-api
```

---

## 53. Separación por ambiente

No mezcles servicios productivos y de desarrollo en el mismo servidor si puedes evitarlo.

Si existen varios ambientes:

```text
dev
staging
production
```

Usa nombres claros:

```text
myapp-api-dev.service
myapp-api-staging.service
myapp-api-prod.service
```

Mejor aún:

```text
Separar servidores por ambiente.
Separar usuarios.
Separar secretos.
Separar bases de datos.
```

---

## 54. Gestión de cambios

Antes de cambiar un servicio:

```text
1. Revisar unit actual.
2. Crear backup o usar Git.
3. Aplicar cambio en staging.
4. Validar config.
5. Reiniciar/recargar.
6. Revisar logs.
7. Ejecutar smoke test.
8. Documentar cambio.
```

Comandos:

```bash
systemctl cat myapp > myapp.service.backup
sudo systemctl daemon-reload
sudo systemctl restart myapp
journalctl -u myapp -n 100
```

---

## 55. Auditoría

Preguntas de auditoría:

```text
¿Qué servicios están habilitados?
¿Qué servicios están fallando?
¿Qué servicios corren como root?
¿Qué servicios tienen acceso a secretos?
¿Qué servicios no tienen límites?
¿Qué servicios no tienen logs?
¿Qué servicios no tienen propietario?
```

Comandos:

```bash
systemctl list-units --type=service
systemctl list-unit-files --type=service
systemctl --failed
```

Buscar servicios propios:

```bash
ls -lah /etc/systemd/system/*.service
```

Ver usuarios:

```bash
ps -eo user,pid,comm,args | grep myapp
```

---

# Parte XII: monitoreo

## 56. Qué monitorear

Para cada servicio:

```text
Estado: activo/inactivo/fallido.
Reinicios.
CPU.
Memoria.
Disco.
Logs de error.
Latencia si expone API.
Health endpoint si existe.
Colas pendientes si es worker.
Tiempo desde última ejecución si es timer.
```

---

## 57. Comandos de monitoreo local

Estado:

```bash
systemctl status myapp
```

Uso de recursos:

```bash
systemctl status myapp
systemctl show myapp -p MemoryCurrent -p CPUUsageNSec -p TasksCurrent
```

Procesos:

```bash
ps aux | grep myapp
```

Árbol de cgroups:

```bash
systemd-cgls
```

Uso de recursos por cgroup:

```bash
systemd-cgtop
```

Logs:

```bash
journalctl -u myapp -f
```

---

## 58. Alertas

Alertas mínimas:

```text
Servicio fallido.
Servicio reiniciándose muchas veces.
Uso de memoria cerca de MemoryMax.
Disco bajo.
Errores frecuentes en logs.
Health check fallido.
Timer sin ejecutar.
```

Herramientas:

```text
Prometheus node_exporter.
systemd exporter.
Netdata.
Zabbix.
Icinga/Nagios.
Grafana Agent.
Uptime Kuma.
CloudWatch Agent en AWS.
Datadog/New Relic.
```

---

## 59. Smoke test después de restart

Para APIs:

```bash
curl -f http://127.0.0.1:8000/health
```

Script:

```bash
#!/bin/bash
set -euo pipefail

sudo systemctl restart myapi
sleep 2
curl -fsS http://127.0.0.1:8000/health
journalctl -u myapi -n 50 --no-pager
```

Uso:

```bash
bash smoke_myapi.sh
```

---

# Parte XIII: despliegue y rollback

## 60. Patrón de releases con symlink

Estructura:

```text
/opt/myapp/
    releases/
        20260704_120000/
        20260704_130000/
    current -> releases/20260704_130000
    shared/
```

Servicio:

```ini
[Service]
WorkingDirectory=/opt/myapp/current
ExecStart=/opt/myapp/current/.venv/bin/python -m myapp
```

Deploy:

```bash
sudo -u myapp mkdir -p /opt/myapp/releases/20260704_130000
sudo -u myapp rsync -a ./dist/ /opt/myapp/releases/20260704_130000/
sudo -u myapp ln -sfn /opt/myapp/releases/20260704_130000 /opt/myapp/current
sudo systemctl restart myapp
```

Rollback:

```bash
sudo -u myapp ln -sfn /opt/myapp/releases/20260704_120000 /opt/myapp/current
sudo systemctl restart myapp
```

---

## 61. Pre y post deploy

Pre deploy:

```text
Validar artifact.
Instalar dependencias.
Validar configuración.
Verificar permisos.
Ejecutar migraciones si aplica.
```

Post deploy:

```text
Reiniciar servicio.
Ver status.
Revisar logs.
Ejecutar smoke test.
Monitorear por algunos minutos.
```

Comandos:

```bash
sudo systemctl restart myapp
sudo systemctl status myapp
journalctl -u myapp -n 100 --no-pager
```

---

## 62. Versionar unit files

Recomendación:

```text
Guardar unit files en Git.
Guardar scripts de deploy en Git.
Guardar documentación de configuración en Git.
No guardar secretos en Git.
```

Ejemplo de repo:

```text
infra/
    systemd/
        myapp-api.service
        myapp-worker.service
        myapp-backup.timer
    scripts/
        deploy_myapp.sh
        smoke_myapp.sh
```

---

# Parte XIV: plantillas de servicios

## 63. Servicios template

Puedes crear servicios parametrizados con `@`.

Archivo:

```bash
sudo nano /etc/systemd/system/myworker@.service
```

Contenido:

```ini
[Unit]
Description=My worker instance %i
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
Environment=WORKER_NAME=%i
ExecStart=/opt/myapp/.venv/bin/python -m myapp.worker --name %i
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Iniciar instancias:

```bash
sudo systemctl start myworker@email
sudo systemctl start myworker@reports
sudo systemctl enable myworker@email
sudo systemctl enable myworker@reports
```

Ver logs:

```bash
journalctl -u myworker@email
```

Ventaja:

```text
Misma definición, múltiples instancias.
```

---

## 64. Environment por instancia

Puedes usar archivo por instancia:

```ini
EnvironmentFile=/etc/myapp/%i.env
```

Archivos:

```text
/etc/myapp/email.env
/etc/myapp/reports.env
```

Ejemplo:

```env
QUEUE_NAME=email
CONCURRENCY=4
```

---

# Parte XV: slices y agrupación de recursos

## 65. Qué es un slice

Un `.slice` agrupa servicios para aplicar recursos comunes.

Ejemplo:

```text
myapp.slice
    myapp-api.service
    myapp-worker.service
```

Crear slice:

```bash
sudo nano /etc/systemd/system/myapp.slice
```

Contenido:

```ini
[Unit]
Description=My App service group

[Slice]
CPUQuota=200%
MemoryMax=2G
```

Asignar servicio al slice:

```ini
[Service]
Slice=myapp.slice
```

Aplicar:

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp-api
sudo systemctl restart myapp-worker
```

Uso:

```text
Limitar todos los componentes de una aplicación juntos.
Separar recursos por producto o tenant.
Gobernanza de recursos.
```

---

# Parte XVI: troubleshooting

## 66. Servicio no arranca

Comandos:

```bash
sudo systemctl status myapp
journalctl -u myapp -n 100 --no-pager
sudo systemctl cat myapp
systemd-analyze verify /etc/systemd/system/myapp.service
```

Causas frecuentes:

```text
ExecStart apunta a ruta inexistente.
Permisos insuficientes.
Usuario no existe.
WorkingDirectory no existe.
Variable de entorno faltante.
Python venv inexistente.
Puerto ocupado.
Error de aplicación.
```

---

## 67. Error: Failed at step EXEC

Significa que systemd no pudo ejecutar el binario.

Revisar:

```bash
ls -l /ruta/del/binario
file /ruta/del/binario
```

Causas:

```text
Ruta incorrecta.
Archivo no ejecutable.
Shebang incorrecto.
No existe intérprete.
Permisos.
```

Solución típica:

```bash
chmod +x /opt/myapp/bin/start.sh
```

Ver shebang:

```bash
head -n 1 /opt/myapp/bin/start.sh
```

Ejemplo correcto:

```bash
#!/bin/bash
```

---

## 68. Error: WorkingDirectory does not exist

Revisar:

```bash
ls -ld /opt/myapp
```

Crear:

```bash
sudo mkdir -p /opt/myapp
sudo chown myapp:myapp /opt/myapp
```

---

## 69. Servicio se reinicia en loop

Ver:

```bash
systemctl status myapp
journalctl -u myapp -f
```

Causas:

```text
La app falla inmediatamente.
Falta configuración.
Puerto ocupado.
Dependencia no disponible.
Permisos.
Restart demasiado agresivo.
```

Temporalmente puedes detener:

```bash
sudo systemctl stop myapp
```

Resetear failed:

```bash
sudo systemctl reset-failed myapp
```

---

## 70. Puerto ocupado

Ver:

```bash
sudo ss -ltnp | grep ':8000'
```

O:

```bash
sudo lsof -i :8000
```

Soluciones:

```text
Cambiar puerto.
Detener proceso que ocupa puerto.
Evitar correr dos instancias del mismo servicio.
```

---

## 71. No aparecen logs

Revisar:

```bash
journalctl -u myapp
systemctl status myapp
```

Causas:

```text
Servicio no inicia.
Proceso escribe a archivo distinto.
StandardOutput=null.
Logging de app mal configurado.
Journal sin persistencia y logs antiguos se perdieron.
```

Asegura:

```ini
StandardOutput=journal
StandardError=journal
```

---

## 72. Variables de entorno no cargan

Ver entorno efectivo no siempre muestra valores sensibles, pero puedes revisar unit:

```bash
systemctl cat myapp
```

Revisar archivo:

```bash
sudo ls -l /etc/myapp/myapp.env
sudo cat /etc/myapp/myapp.env
```

Causas:

```text
EnvironmentFile con ruta incorrecta.
Permisos impiden lectura.
Sintaxis inválida.
Olvidaste daemon-reload/restart.
```

Formato correcto:

```env
KEY=value
LOG_LEVEL=INFO
```

Evita:

```env
export KEY=value
```

---

## 73. Cambios no se aplican

Checklist:

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp
systemctl cat myapp
systemctl show myapp -p FragmentPath -p DropInPaths
```

Causas:

```text
No ejecutaste daemon-reload.
Hay override que pisa configuración.
Editaste otro archivo.
El servicio no se reinició.
```

---

# Parte XVII: ejemplos completos

## 74. Servicio API Python productivo

```ini
[Unit]
Description=Billing API
Documentation=https://wiki.example.com/billing-api
After=network-online.target
Wants=network-online.target
StartLimitIntervalSec=300
StartLimitBurst=5

[Service]
Type=simple
User=billing
Group=billing
WorkingDirectory=/opt/billing/current
EnvironmentFile=/etc/billing/billing.env

ExecStart=/opt/billing/current/.venv/bin/uvicorn billing.main:app --host 127.0.0.1 --port 8000
ExecReload=/bin/kill -HUP $MAINPID

Restart=on-failure
RestartSec=10
TimeoutStopSec=30
KillSignal=SIGTERM

StandardOutput=journal
StandardError=journal

MemoryHigh=768M
MemoryMax=1G
CPUQuota=200%
TasksMax=256
LimitNOFILE=65535

NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/billing /var/log/billing /run/billing

[Install]
WantedBy=multi-user.target
```

---

## 75. Worker Python

```ini
[Unit]
Description=Billing worker
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=billing
Group=billing
WorkingDirectory=/opt/billing/current
EnvironmentFile=/etc/billing/billing.env
ExecStart=/opt/billing/current/.venv/bin/python -m billing.worker

Restart=always
RestartSec=10

MemoryMax=512M
CPUQuota=100%
TasksMax=128

NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/billing /var/log/billing

[Install]
WantedBy=multi-user.target
```

---

## 76. Job recurrente con timer

Servicio:

```ini
[Unit]
Description=Billing report generator

[Service]
Type=oneshot
User=billing
Group=billing
WorkingDirectory=/opt/billing/current
EnvironmentFile=/etc/billing/billing.env
ExecStart=/opt/billing/current/.venv/bin/python -m billing.generate_reports
```

Timer:

```ini
[Unit]
Description=Run billing report generator every day

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true
Unit=billing-reports.service

[Install]
WantedBy=timers.target
```

Activar:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now billing-reports.timer
systemctl list-timers
```

---

# Parte XVIII: checklists

## 77. Checklist de creación de servicio

```text
[ ] Usuario de sistema creado.
[ ] Directorios creados.
[ ] Permisos asignados.
[ ] App probada manualmente con el usuario de servicio.
[ ] Unit file creado en /etc/systemd/system.
[ ] systemd-analyze verify ejecutado.
[ ] daemon-reload ejecutado.
[ ] Servicio iniciado.
[ ] Servicio habilitado al boot.
[ ] Logs revisados con journalctl.
[ ] Smoke test ejecutado.
```

---

## 78. Checklist de seguridad

```text
[ ] No corre como root.
[ ] Tiene usuario dedicado.
[ ] NoNewPrivileges=true.
[ ] PrivateTmp=true si aplica.
[ ] ProtectSystem configurado si aplica.
[ ] ProtectHome configurado si aplica.
[ ] ReadWritePaths limitado.
[ ] EnvironmentFile con permisos restrictivos.
[ ] Secretos no están en Git.
[ ] systemd-analyze security revisado.
```

---

## 79. Checklist de recursos

```text
[ ] MemoryMax definido.
[ ] CPUQuota definido si aplica.
[ ] TasksMax definido.
[ ] LimitNOFILE definido si hay muchas conexiones.
[ ] Restart policy definida.
[ ] StartLimitBurst definido para evitar loops.
[ ] Logs tienen retención.
[ ] Disco monitoreado.
```

---

## 80. Checklist de monitoreo

```text
[ ] systemctl status muestra servicio activo.
[ ] journalctl muestra logs útiles.
[ ] Health check definido si aplica.
[ ] Métrica de estado activa/fallida.
[ ] Métrica de reinicios.
[ ] Métrica de CPU.
[ ] Métrica de memoria.
[ ] Métrica de disco.
[ ] Alertas configuradas.
[ ] Runbook documentado.
```

---

## 81. Checklist de gobernanza

```text
[ ] Servicio tiene propietario.
[ ] Servicio tiene propósito documentado.
[ ] Servicio tiene criticidad definida.
[ ] Servicio tiene ambiente claro.
[ ] Servicio tiene dependencias documentadas.
[ ] Servicio tiene procedimiento de deploy.
[ ] Servicio tiene rollback.
[ ] Servicio tiene configuración versionada.
[ ] Servicio tiene política de logs.
[ ] Servicio tiene inventario actualizado.
```

---

# Parte XIX: comandos rápidos

## 82. systemctl

```bash
sudo systemctl start myapp
sudo systemctl stop myapp
sudo systemctl restart myapp
sudo systemctl reload myapp
sudo systemctl status myapp
sudo systemctl enable myapp
sudo systemctl disable myapp
sudo systemctl daemon-reload
sudo systemctl reset-failed myapp
systemctl is-active myapp
systemctl is-enabled myapp
systemctl --failed
systemctl list-units --type=service
systemctl list-unit-files --type=service
systemctl cat myapp
systemctl show myapp
```

---

## 83. journalctl

```bash
journalctl -u myapp
journalctl -u myapp -f
journalctl -u myapp -n 100
journalctl -u myapp -b
journalctl -u myapp --since "1 hour ago"
journalctl -u myapp -p err
journalctl -u myapp -o short-iso
journalctl --disk-usage
sudo journalctl --vacuum-time=30d
```

---

## 84. análisis y recursos

```bash
systemd-analyze verify /etc/systemd/system/myapp.service
systemd-analyze security myapp.service
systemd-cgls
systemd-cgtop
systemctl show myapp -p MemoryCurrent -p CPUUsageNSec -p TasksCurrent
systemctl list-timers
```

---

# Parte XX: resumen de reglas principales

```text
1. Crea servicios propios en /etc/systemd/system.
2. Usa usuarios dedicados, no root.
3. Ejecuta daemon-reload después de cambiar unit files.
4. Usa systemctl para ciclo de vida.
5. Usa journalctl para logs.
6. Escribe logs a stdout/stderr salvo razón para usar archivos.
7. Define Restart y RestartSec.
8. Evita loops con StartLimitBurst.
9. Define MemoryMax, CPUQuota y TasksMax cuando el servicio sea relevante.
10. Usa EnvironmentFile para configuración, pero protege permisos.
11. No guardes secretos en Git.
12. Usa timers para jobs recurrentes.
13. Usa hardening progresivo: NoNewPrivileges, PrivateTmp, ProtectSystem.
14. Valida con systemd-analyze verify.
15. Evalúa seguridad con systemd-analyze security.
16. Documenta propietario, propósito, dependencias y runbook.
17. Versiona unit files.
18. Monitorea estado, logs, CPU, memoria, disco y reinicios.
19. Ejecuta smoke tests después de reiniciar o desplegar.
20. Diseña rollback antes de tocar producción.
```

---

# Fuentes de referencia recomendadas

```text
- Ubuntu Manpages: systemctl.
- Ubuntu Manpages: journalctl.
- Ubuntu documentation: systemd overview.
- systemd manual: systemd.service.
- systemd manual: systemd.exec.
- systemd manual: systemd.resource-control.
- systemd manual: systemd.timer.
- systemd manual: systemd.journald.
- systemd-analyze manual.
```
