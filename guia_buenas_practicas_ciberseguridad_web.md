# Guía de buenas prácticas de ciberseguridad para desarrollo web

## Objetivo

Esta guía resume buenas prácticas de ciberseguridad aplicadas al desarrollo web, backend, frontend y APIs. Está pensada para equipos que construyen aplicaciones reales con usuarios, sesiones, bases de datos, archivos, integraciones externas y despliegues en la nube.

La guía está organizada en cuatro niveles:

```text
1. Fundamentos: principios, amenazas comunes y mentalidad defensiva.
2. Desarrollo seguro: autenticación, autorización, sesiones, backend, frontend y APIs.
3. Riesgos típicos: SQL injection, XSS, CSRF, IDOR, SSRF, cookies, secretos, dependencias y supply chain.
4. Auditoría y operación: revisión de seguridad, pruebas, hardening, observabilidad e incidentes.
```

El objetivo no es enseñar a atacar sistemas ajenos. Los ejemplos se entregan para reconocer fallas, corregirlas y auditar aplicaciones propias o autorizadas.

---

# Parte I: fundamentos

## 1. Principios base

Una aplicación segura no depende de una sola defensa. Debe combinar varias capas.

Principios principales:

```text
Defensa en profundidad.
Mínimo privilegio.
Validar en servidor.
No confiar en el cliente.
Fallar de forma segura.
Reducir superficie de ataque.
Separar responsabilidades.
Registrar eventos críticos.
Proteger secretos.
Mantener dependencias actualizadas.
```

Regla central:

```text
El backend debe ser la autoridad.
El frontend puede mejorar experiencia de usuario, pero no debe ser la fuente de verdad de permisos, precios, roles, límites, identidad o reglas críticas.
```

Ejemplo:

```text
Frontend:
- Oculta botón "Eliminar usuario" si el usuario no es admin.

Backend:
- Verifica en cada request si el usuario realmente tiene permiso para eliminar ese usuario.
```

El primer punto mejora UX. El segundo protege el sistema.

---

## 2. Modelo mental de amenaza

Antes de escribir código, identifica:

```text
1. Qué activos proteges.
2. Quiénes son los usuarios.
3. Qué datos son sensibles.
4. Qué acciones son críticas.
5. Qué integraciones externas existen.
6. Qué puede controlar un atacante.
7. Qué logs necesitas para investigar incidentes.
```

Activos típicos:

```text
Credenciales.
Tokens de sesión.
Datos personales.
Datos financieros.
Datos médicos.
Archivos privados.
Permisos de administración.
API keys.
Modelos de IA o datos de entrenamiento.
Propiedad intelectual.
```

Preguntas útiles:

```text
¿Qué pasa si alguien modifica este ID?
¿Qué pasa si alguien repite este request 10.000 veces?
¿Qué pasa si el token se filtra?
¿Qué pasa si el frontend es modificado?
¿Qué pasa si una dependencia se compromete?
¿Qué pasa si un usuario intenta acceder a datos de otro tenant?
```

---

## 3. Riesgos web más frecuentes

Riesgos comunes:

```text
Control de acceso roto.
Errores criptográficos.
Inyecciones.
Diseño inseguro.
Mala configuración.
Componentes vulnerables.
Fallos de identificación y autenticación.
Fallos de integridad de software y datos.
Logging y monitoreo insuficientes.
Server-Side Request Forgery, SSRF.
```

Estos riesgos están alineados con las familias más conocidas de OWASP Top 10 para aplicaciones web.

---

## 4. Seguridad no es solo código

La seguridad depende de:

```text
Código.
Configuración.
Infraestructura.
Secretos.
Dependencias.
Procesos de despliegue.
Observabilidad.
Control de acceso interno.
Gestión de incidentes.
Cultura del equipo.
```

Una aplicación puede tener buen código y seguir siendo insegura por:

```text
Variables de entorno filtradas.
Bucket público.
CORS abierto.
Base de datos expuesta a internet.
Credenciales reutilizadas.
Dependencias abandonadas.
Logs con tokens.
Permisos excesivos en cloud.
```

---

# Parte II: autenticación

## 5. Autenticación vs autorización

No son lo mismo.

```text
Autenticación:
Demuestra quién eres.

Autorización:
Determina qué puedes hacer.
```

Ejemplo:

```text
Usuario autenticado:
felipe@example.com inició sesión correctamente.

Autorización:
felipe@example.com puede leer sus propios documentos,
pero no eliminar usuarios del sistema.
```

Una aplicación puede autenticar bien y autorizar mal.

---

## 6. Reglas de autenticación

Buenas prácticas:

```text
Usa HTTPS siempre.
No guardes contraseñas en texto plano.
Usa hash de contraseña resistente: Argon2id, bcrypt o scrypt.
Agrega MFA cuando el riesgo lo amerite.
Implementa rate limiting en login.
Evita mensajes que revelen si un email existe.
Regenera sesión al iniciar sesión.
Permite cerrar sesiones activas.
Registra logins sospechosos.
No envíes contraseñas por email.
No guardes tokens en logs.
```

Mensaje menos recomendable:

```text
El email no existe.
```

Mensaje más seguro:

```text
Credenciales inválidas.
```

Esto evita enumeración de usuarios.

---

## 7. Contraseñas

Recomendaciones:

```text
Permite contraseñas largas.
No fuerces rotación periódica sin causa.
No uses reglas artificiales excesivas como "exactamente una mayúscula, un número y un símbolo".
Bloquea contraseñas filtradas o muy comunes.
Usa MFA para cuentas sensibles.
No guardes pistas de contraseña.
```

Hash recomendado, ejemplo conceptual con Python:

```python
from passlib.context import CryptContext

pwd_context = CryptContext(
    schemes=["argon2", "bcrypt"],
    deprecated="auto",
)

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(password: str, password_hash: str) -> bool:
    return pwd_context.verify(password, password_hash)
```

Nunca hagas esto:

```python
# Mala práctica
user.password = plain_password
```

Tampoco uses hashes rápidos para contraseñas:

```text
MD5
SHA-1
SHA-256 directo
```

Estos algoritmos son útiles para integridad, pero no para almacenar contraseñas de usuarios.

---

## 8. MFA

MFA reduce el riesgo de cuentas comprometidas.

Tipos comunes:

```text
TOTP: códigos temporales en app autenticadora.
WebAuthn/FIDO2: llaves de seguridad o passkeys.
Push: aprobación en dispositivo.
SMS: mejor que nada, pero menos robusto.
```

Buenas prácticas:

```text
Prefiere WebAuthn/FIDO2 para cuentas críticas.
Permite códigos de recuperación.
Protege el flujo de enrolamiento de MFA.
Pide reautenticación para desactivar MFA.
Registra cambios de MFA.
```

---

## 9. Sesiones

Una sesión representa el estado autenticado de un usuario.

Buenas prácticas:

```text
Usa identificadores de sesión aleatorios y largos.
Regenera la sesión después de login.
Invalida sesión en logout.
Define expiración absoluta y por inactividad.
Permite revocar sesiones.
No incluyas datos sensibles en el ID de sesión.
No expongas session IDs en URLs.
```

Evita:

```http
GET /dashboard?session_id=abc123
```

Mejor:

```text
Cookie segura enviada automáticamente por el navegador.
```

---

## 10. Cookies seguras

Para sesiones web tradicionales, una cookie segura suele ser una opción sólida si está bien configurada.

Ejemplo HTTP:

```http
Set-Cookie: session=opaque_random_value; Path=/; HttpOnly; Secure; SameSite=Lax; Max-Age=3600
```

Atributos importantes:

```text
HttpOnly:
Impide que JavaScript lea la cookie mediante document.cookie.

Secure:
Solo envía la cookie por HTTPS.

SameSite:
Reduce riesgo de CSRF en requests cross-site.

Path:
Limita dónde se envía la cookie.

Max-Age / Expires:
Define expiración.

__Host- prefix:
Endurece cookies de sesión si se usa con Secure, Path=/ y sin Domain.
```

Ejemplo más estricto:

```http
Set-Cookie: __Host-session=opaque_random_value; Path=/; HttpOnly; Secure; SameSite=Lax; Max-Age=3600
```

Regla:

```text
Una cookie HttpOnly ayuda contra robo por XSS,
pero no elimina el riesgo de que un XSS haga requests autenticados desde el navegador de la víctima.
```

Por eso también necesitas:

```text
Prevención de XSS.
CSRF tokens cuando corresponda.
SameSite.
CSP.
Validaciones backend.
```

---

## 11. LocalStorage, SessionStorage y cookies

Comparación práctica:

```text
Cookie HttpOnly:
- JavaScript no puede leerla.
- El navegador la envía automáticamente.
- Requiere defensa CSRF si se usa para autenticación.

localStorage:
- JavaScript puede leerlo.
- Vulnerable al robo de tokens si hay XSS.
- No se envía automáticamente.

sessionStorage:
- Similar a localStorage, pero por pestaña/sesión.
- También accesible desde JavaScript.
```

Recomendación general:

```text
Para sesiones web: cookie HttpOnly + Secure + SameSite.
Para clientes no navegador: Authorization: Bearer <token>.
Para SPAs: evalúa BFF, sesiones con cookies HttpOnly o flujos OAuth/OIDC con PKCE.
```

BFF significa Backend For Frontend. En ese patrón, el frontend no maneja directamente tokens sensibles; habla con un backend propio que mantiene sesión de forma segura.

---

## 12. JWT

JWT no es una solución mágica. Es un formato de token.

Buenas prácticas:

```text
Usa JWT solo si necesitas tokens autocontenidos o interoperabilidad.
Firma siempre los tokens.
Verifica firma, expiración, issuer, audience y algoritmo.
Usa expiraciones cortas para access tokens.
No guardes secretos en el payload.
No confíes en el contenido sin verificar.
Implementa revocación o listas de invalidez si el riesgo lo exige.
Rota claves.
No aceptes algoritmo "none".
```

Payload:

```json
{
    "sub": "user_123",
    "iss": "https://auth.example.com",
    "aud": "api.example.com",
    "exp": 1735689600,
    "scope": "documents:read"
}
```

No incluyas:

```json
{
    "password": "secret",
    "credit_card": "4111111111111111"
}
```

Regla:

```text
Un JWT firmado es legible, no necesariamente cifrado.
```

---

## 13. OAuth2 y OpenID Connect

OAuth2 es un framework de autorización. OpenID Connect agrega identidad sobre OAuth2.

Uso típico:

```text
OAuth2:
Autorizar acceso a un recurso.

OpenID Connect:
Iniciar sesión y obtener identidad del usuario.
```

Flujo recomendado para aplicaciones modernas:

```text
Authorization Code Flow con PKCE.
```

Evita flujos obsoletos o menos seguros:

```text
Implicit flow.
Resource Owner Password Credentials, salvo casos heredados muy controlados.
```

Conceptos:

```text
Authorization server:
Servidor que autentica y emite tokens.

Resource server:
API que recibe y valida access tokens.

Client:
Aplicación que solicita acceso.

Access token:
Token para llamar APIs.

Refresh token:
Token para obtener nuevos access tokens.

Scope:
Permiso solicitado.

Audience:
API destino del token.
```

Reglas prácticas:

```text
Usa PKCE para clientes públicos y SPAs.
Valida redirect_uri estrictamente.
Usa state para prevenir CSRF en el flujo.
Usa nonce en OpenID Connect.
No pongas tokens en URLs.
No mezcles tokens de una audience con otra.
Rota refresh tokens.
Revoca tokens al cerrar sesión cuando aplique.
```

---

# Parte III: autorización y modelos de permisos

## 14. El backend como autoridad

El frontend puede ocultar opciones, pero el backend debe verificar permisos.

Mala práctica:

```javascript
if (user.role === "admin") {
    showDeleteButton();
}
```

y en backend:

```python
@app.delete("/users/{user_id}")
def delete_user(user_id: int):
    delete_user_from_db(user_id)
    return {"status": "ok"}
```

El botón se oculta, pero cualquiera podría llamar el endpoint si conoce la URL.

Mejor:

```python
@app.delete("/users/{user_id}")
def delete_user(
    user_id: int,
    current_user: User = Depends(get_current_user),
):
    if not current_user.is_admin:
        raise HTTPException(status_code=403, detail="Forbidden")

    delete_user_from_db(user_id)
    return Response(status_code=204)
```

---

## 15. Modelos de autorización

### 15.1 RBAC

Role-Based Access Control.

```text
Usuario -> Roles -> Permisos
```

Ejemplo:

```text
admin:
- users:read
- users:delete
- documents:read
- documents:write

editor:
- documents:read
- documents:write

viewer:
- documents:read
```

Ventajas:

```text
Simple.
Fácil de explicar.
Útil para sistemas medianos.
```

Riesgos:

```text
Roles demasiado amplios.
Roles duplicados.
Explosión de roles.
```

---

### 15.2 ABAC

Attribute-Based Access Control.

Decide según atributos.

```text
Usuario.department == Documento.department
Usuario.clearance >= Documento.sensitivity
Request.ip in allowed_ranges
Hora dentro de horario laboral
```

Ejemplo conceptual:

```python
def can_read_document(user: User, document: Document) -> bool:
    if user.is_admin:
        return True

    if user.department != document.department:
        return False

    return user.clearance_level >= document.sensitivity_level
```

Ventajas:

```text
Flexible.
Expresa reglas de negocio complejas.
Útil en empresas grandes.
```

Riesgos:

```text
Más difícil de auditar.
Requiere buenas pruebas.
```

---

### 15.3 ReBAC

Relationship-Based Access Control.

Decide según relaciones.

```text
Usuario es dueño del documento.
Usuario pertenece al equipo del proyecto.
Usuario fue invitado a la carpeta.
```

Ejemplo:

```python
def can_edit_project(user: User, project: Project) -> bool:
    return user.id in project.editor_user_ids
```

Útil en:

```text
Google Drive.
Notion.
Sistemas colaborativos.
CRMs.
Proyectos compartidos.
```

---

### 15.4 Permisos por recurso

No basta con verificar el rol global. Hay que verificar propiedad o relación con el recurso.

Endpoint vulnerable:

```python
@app.get("/documents/{document_id}")
def get_document(document_id: int):
    return db.get_document(document_id)
```

Mejor:

```python
@app.get("/documents/{document_id}")
def get_document(
    document_id: int,
    current_user: User = Depends(get_current_user),
):
    document = db.get_document(document_id)

    if document is None:
        raise HTTPException(status_code=404, detail="Document not found")

    if document.owner_id != current_user.id and not current_user.is_admin:
        raise HTTPException(status_code=403, detail="Forbidden")

    return document
```

---

## 16. IDOR y BOLA

IDOR significa Insecure Direct Object Reference. BOLA significa Broken Object Level Authorization.

Ejemplo vulnerable:

```http
GET /invoices/1001
GET /invoices/1002
GET /invoices/1003
```

Si un usuario cambia el ID y ve facturas ajenas, hay una falla de autorización.

Mitigaciones:

```text
Verifica permisos por objeto en cada acceso.
No confíes en IDs enviados por cliente.
Usa filtros por owner_id o tenant_id en consultas.
Implementa pruebas automatizadas de acceso cruzado.
Registra intentos denegados.
```

Consulta más segura:

```sql
SELECT *
FROM invoices
WHERE id = $1
  AND owner_id = $2;
```

No:

```sql
SELECT *
FROM invoices
WHERE id = $1;
```

---

## 17. Multi-tenant

En sistemas multi-tenant, cada request debe estar acotado a un tenant.

Reglas:

```text
Todo recurso sensible debe tener tenant_id o una relación equivalente.
El tenant_id no debe confiarse desde el frontend sin validación.
Las consultas deben filtrar por tenant_id.
Las tareas batch también deben respetar tenant_id.
Los logs deben incluir tenant_id, pero no datos sensibles.
```

Ejemplo:

```sql
SELECT *
FROM documents
WHERE id = $1
  AND tenant_id = $2;
```

Riesgo común:

```text
El endpoint filtra por tenant_id,
pero un job, exportación o panel admin no lo hace.
```

---

# Parte IV: backend seguro

## 18. Validación de entrada

Todo dato externo es no confiable:

```text
Body JSON.
Query params.
Path params.
Headers.
Cookies.
Archivos.
Webhooks.
Mensajes de cola.
Datos de APIs externas.
Datos de IA generativa.
```

Buenas prácticas:

```text
Valida tipos.
Valida rangos.
Valida tamaño máximo.
Valida formato.
Normaliza antes de comparar.
Rechaza campos inesperados si el caso lo requiere.
No confíes en validación del frontend.
```

Ejemplo con Pydantic:

```python
from pydantic import BaseModel, Field, HttpUrl

class CreateDocumentRequest(BaseModel):
    title: str = Field(min_length=1, max_length=120)
    source_url: HttpUrl | None = None
    visibility: str = Field(pattern="^(private|team|public)$")
```

---

## 19. Output encoding

Validar entrada no reemplaza escapar salida. Para prevenir XSS, el contexto importa.

Contextos distintos:

```text
HTML.
Atributos HTML.
JavaScript.
CSS.
URL.
Markdown renderizado.
```

Regla:

```text
Codifica la salida según el contexto donde será insertada.
```

Ejemplo seguro con template engine que escapa por defecto:

```html
<p>{{ user_display_name }}</p>
```

Peligroso:

```html
<div>
    {{ raw_html | safe }}
</div>
```

Solo usa HTML sin escapar cuando:

```text
El contenido fue sanitizado.
La fuente es confiable.
El caso está documentado.
Hay pruebas.
```

---

## 20. SQL injection

SQL injection ocurre cuando datos del usuario se mezclan con SQL como texto ejecutable.

Mala práctica:

```python
query = f"SELECT * FROM users WHERE email = '{email}'"
cursor.execute(query)
```

Entrada maliciosa podría alterar la consulta.

Mejor:

```python
query = "SELECT * FROM users WHERE email = %s"
cursor.execute(query, (email,))
```

Con SQLAlchemy:

```python
stmt = select(User).where(User.email == email)
result = session.execute(stmt).scalar_one_or_none()
```

Con SQLModel:

```python
statement = select(User).where(User.email == email)
user = session.exec(statement).first()
```

Buenas prácticas:

```text
Usa consultas parametrizadas.
Usa ORM correctamente.
Evita concatenar strings SQL.
Valida allowlists para nombres de columnas u ordenamiento.
No uses permisos de superusuario para la app.
Limita privilegios de la cuenta de base de datos.
Registra errores sin exponer SQL interno al usuario.
```

Ordenamiento seguro:

```python
allowed_sort_fields = {
    "created_at": User.created_at,
    "email": User.email,
}

sort_column = allowed_sort_fields.get(sort)

if sort_column is None:
    raise HTTPException(status_code=400, detail="Invalid sort field")

statement = select(User).order_by(sort_column)
```

No parametrices identificadores como si fueran valores; usa allowlist.

---

## 21. NoSQL injection

También existe inyección en bases NoSQL.

Mala práctica conceptual:

```javascript
db.users.findOne({
    email: req.body.email,
    password: req.body.password
});
```

Si el backend acepta objetos arbitrarios, un atacante podría enviar operadores inesperados.

Mitigaciones:

```text
Valida tipos estrictamente.
Rechaza objetos donde esperas strings.
Usa esquemas.
No pases directamente req.body al query.
Usa allowlists.
```

---

## 22. Command injection

Ocurre cuando datos del usuario llegan a comandos del sistema.

Mala práctica:

```python
os.system(f"convert {filename} output.pdf")
```

Mejor:

```python
subprocess.run(
    ["convert", input_path, output_path],
    check=True,
    timeout=30,
)
```

Reglas:

```text
Evita shell=True.
Usa arrays de argumentos.
Valida rutas.
Usa timeouts.
Ejecuta procesos con usuario sin privilegios.
Aísla procesamiento riesgoso.
```

---

## 23. Path traversal

Riesgo:

```http
GET /files?name=../../etc/passwd
```

Mala práctica:

```python
path = f"/app/uploads/{filename}"
return FileResponse(path)
```

Mejor:

```python
from pathlib import Path

BASE_DIR = Path("/app/uploads").resolve()

def safe_path(filename: str) -> Path:
    candidate = (BASE_DIR / filename).resolve()

    if not str(candidate).startswith(str(BASE_DIR)):
        raise HTTPException(status_code=400, detail="Invalid file path")

    return candidate
```

Buenas prácticas:

```text
No uses rutas arbitrarias del usuario.
Normaliza y verifica ruta final.
Usa IDs de archivo en vez de nombres directos.
Guarda metadata en base de datos.
Controla permisos por archivo.
```

---

## 24. SSRF

Server-Side Request Forgery ocurre cuando el servidor hace requests a URLs controladas por el usuario y puede alcanzar redes internas.

Ejemplo de funcionalidad riesgosa:

```text
"Ingresa una URL y descargaremos el contenido."
```

Mitigaciones:

```text
Usa allowlist de dominios si es posible.
Bloquea IPs privadas, loopback y metadatos cloud.
Resuelve DNS y valida la IP final.
Revalida después de redirecciones.
Limita protocolos a http/https.
Usa timeouts.
Limita tamaño de respuesta.
No envíes credenciales internas.
Ejecuta fetchers en red aislada.
```

Pseudocódigo:

```python
def validate_external_url(url: str) -> None:
    parsed = urlparse(url)

    if parsed.scheme not in {"http", "https"}:
        raise ValueError("Invalid scheme")

    if parsed.hostname not in ALLOWED_HOSTS:
        raise ValueError("Host not allowed")
```

Cuando no puedas usar allowlist, el diseño es más riesgoso y requiere defensas de red.

---

## 25. File uploads

Subir archivos es una fuente frecuente de fallas.

Riesgos:

```text
Malware.
Archivos enormes.
Extensiones engañosas.
Content-Type falso.
Path traversal.
Ejecución de archivos subidos.
XSS por SVG/HTML.
PDFs maliciosos.
ZIP bombs.
```

Buenas prácticas:

```text
Valida tamaño máximo.
Valida extensión y MIME real.
Renombra archivos.
Guarda fuera del webroot.
No ejecutes archivos subidos.
Escanea con antivirus si aplica.
Procesa en sandbox.
Convierte a formatos seguros cuando sea posible.
Aplica permisos por archivo.
Usa URLs firmadas y expirables para descarga privada.
```

Ejemplo de política:

```text
PDF:
- Máximo 20 MB.
- Content-Type esperado.
- Validación de firma/magic bytes.
- Almacenamiento privado.
- Escaneo antivirus.
- Procesamiento asíncrono.
- Timeout.
```

---

## 26. Errores y excepciones

No expongas detalles internos.

Mala respuesta:

```json
{
    "error": "psycopg2.errors.UndefinedTable: relation users_private does not exist",
    "query": "SELECT * FROM users_private"
}
```

Mejor:

```json
{
    "detail": "Internal server error"
}
```

Buenas prácticas:

```text
Muestra mensajes genéricos al usuario.
Registra detalle interno en logs seguros.
Asigna correlation_id a cada request.
No loguees tokens, contraseñas ni datos sensibles.
Distingue errores 400, 401, 403, 404, 409, 422 y 500.
```

---

## 27. Rate limiting y abuso

Protege endpoints sensibles.

Aplicar límites en:

```text
Login.
Registro.
Recuperación de contraseña.
Verificación MFA.
Búsqueda.
Carga de archivos.
Endpoints costosos.
APIs públicas.
Webhooks.
```

Estrategias:

```text
Por IP.
Por usuario.
Por tenant.
Por API key.
Por endpoint.
Por fingerprint de riesgo.
```

Ejemplo conceptual:

```text
POST /login:
- 5 intentos por minuto por IP.
- 10 intentos por hora por cuenta.
- Retraso progresivo.
- Alerta si hay patrón de credential stuffing.
```

---

## 28. Webhooks

Los webhooks son entrada externa. Deben verificarse.

Buenas prácticas:

```text
Verifica firma HMAC o firma asimétrica.
Valida timestamp para evitar replay.
Usa idempotency keys.
Registra event_id.
Procesa de forma asíncrona.
Responde rápido.
No confíes solo en IP allowlist.
```

Ejemplo conceptual:

```python
def verify_signature(payload: bytes, header_signature: str, secret: str) -> bool:
    expected = hmac.new(
        secret.encode(),
        payload,
        hashlib.sha256,
    ).hexdigest()

    return hmac.compare_digest(expected, header_signature)
```

---

# Parte V: frontend seguro

## 29. XSS

Cross-Site Scripting ocurre cuando contenido no confiable se ejecuta como código en el navegador.

Impactos:

```text
Robo de datos accesibles por JavaScript.
Acciones en nombre del usuario.
Modificación del DOM.
Redirecciones.
Captura de formularios.
Bypass visual de controles.
```

Reglas:

```text
No insertes HTML no confiable.
Usa escaping automático del framework.
Sanitiza si necesitas renderizar HTML.
Evita dangerouslySetInnerHTML salvo casos controlados.
No construyas scripts con strings de usuario.
Usa CSP.
```

Mala práctica:

```javascript
element.innerHTML = userInput;
```

Mejor:

```javascript
element.textContent = userInput;
```

Si necesitas HTML enriquecido, sanitiza:

```javascript
import DOMPurify from "dompurify";

preview.innerHTML = DOMPurify.sanitize(markdownHtml);
```

En React:

```jsx
<p>{userInput}</p>
```

Evita:

```jsx
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

---

## 30. Lectura de cookies

JavaScript puede leer cookies que no tienen `HttpOnly`.

Ejemplo:

```javascript
console.log(document.cookie);
```

Por eso una cookie de sesión debe marcarse como `HttpOnly`.

Mala práctica:

```http
Set-Cookie: session=abc123; Path=/; Secure; SameSite=Lax
```

Mejor:

```http
Set-Cookie: session=abc123; Path=/; HttpOnly; Secure; SameSite=Lax
```

Pero recuerda:

```text
HttpOnly impide lectura directa desde JavaScript.
No impide que el navegador envíe la cookie en requests.
No reemplaza CSRF ni prevención de XSS.
```

---

## 31. CSRF

Cross-Site Request Forgery ocurre cuando un sitio externo induce al navegador de la víctima a hacer una request autenticada.

Afecta especialmente a autenticación basada en cookies.

Defensas:

```text
SameSite=Lax o Strict.
CSRF tokens.
Validación Origin/Referer.
Métodos seguros correctamente usados.
No cambiar estado con GET.
Reautenticación para acciones críticas.
```

Mala práctica:

```http
GET /transfer?to=attacker&amount=1000
```

Mejor:

```http
POST /transfer
Content-Type: application/json
X-CSRF-Token: token
```

Regla:

```text
GET no debe modificar estado.
```

---

## 32. CORS

CORS no es autenticación ni autorización. Es una política del navegador para controlar qué orígenes pueden leer respuestas cross-origin.

Mala configuración:

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

Buena práctica:

```text
Define allowlist explícita de orígenes.
Evita credentials si no son necesarias.
No uses "*" con APIs privadas.
No confundas CORS con protección backend.
```

Ejemplo:

```python
allowed_origins = [
    "https://app.example.com",
    "https://admin.example.com",
]
```

---

## 33. CSP

Content Security Policy reduce impacto de XSS controlando qué recursos puede cargar la página.

Ejemplo base:

```http
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'
```

Política más realista con nonces:

```http
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-random-value'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'
```

Buenas prácticas:

```text
Evita 'unsafe-inline' si puedes.
Usa nonces o hashes para scripts inline.
Define frame-ancestors para evitar clickjacking.
Empieza con Content-Security-Policy-Report-Only si necesitas medir impacto.
```

---

## 34. Clickjacking

Clickjacking ocurre cuando tu sitio se carga dentro de un iframe malicioso.

Defensas:

```http
Content-Security-Policy: frame-ancestors 'none'
```

O para permitir solo tu dominio:

```http
Content-Security-Policy: frame-ancestors 'self'
```

También existe:

```http
X-Frame-Options: DENY
```

CSP `frame-ancestors` es más flexible y moderna.

---

## 35. Dependencias frontend

Riesgos:

```text
Paquetes abandonados.
Typosquatting.
Scripts de terceros comprometidos.
Dependencias transitivas vulnerables.
Build scripts maliciosos.
```

Buenas prácticas:

```text
Usa lockfiles.
Revisa cambios mayores.
Audita dependencias.
Elimina paquetes innecesarios.
Fija versiones en producción.
Evita cargar scripts de terceros sin necesidad.
Usa Subresource Integrity para CDN cuando aplique.
```

Ejemplo SRI:

```html
<script
    src="https://cdn.example.com/library.min.js"
    integrity="sha384-..."
    crossorigin="anonymous">
</script>
```

---

# Parte VI: APIs seguras

## 36. Diseño seguro de APIs

Buenas prácticas:

```text
Usa HTTPS.
Autentica endpoints privados.
Autoriza por recurso.
Valida entrada.
Limita tamaño de payload.
Usa paginación.
Aplica rate limiting.
No expongas errores internos.
Versiona APIs.
Documenta con OpenAPI.
Registra correlation_id.
```

Códigos HTTP:

```text
200 OK                  lectura exitosa.
201 Created             recurso creado.
204 No Content           operación exitosa sin cuerpo.
400 Bad Request          solicitud inválida.
401 Unauthorized         falta autenticación válida.
403 Forbidden            autenticado, sin permiso.
404 Not Found            no existe o no conviene revelar existencia.
409 Conflict             conflicto de estado.
422 Unprocessable Entity validación semántica fallida.
429 Too Many Requests    rate limit.
500 Internal Server Error error inesperado.
```

---

## 37. REST

Reglas:

```text
Usa recursos en plural.
No cambies estado con GET.
Usa PATCH para cambios parciales.
Usa PUT para reemplazo completo.
Valida ownership en cada recurso.
Filtra por tenant cuando corresponda.
```

Ejemplo seguro:

```http
GET /api/documents?limit=20&offset=0
POST /api/documents
GET /api/documents/{document_id}
PATCH /api/documents/{document_id}
DELETE /api/documents/{document_id}
```

---

## 38. GraphQL

Riesgos comunes:

```text
Consultas demasiado profundas.
Introspección expuesta en producción sin control.
Errores con detalles internos.
Falta de autorización por campo.
N+1 queries.
Enumeración de datos.
```

Mitigaciones:

```text
Límites de profundidad.
Límites de complejidad.
Timeouts.
Persisted queries.
Autorización por resolver/campo.
Desactivar o restringir introspección.
Rate limiting.
```

---

## 39. API keys

API keys identifican aplicaciones o integraciones. No deberían representar usuarios humanos sin contexto adicional.

Buenas prácticas:

```text
Genera claves largas y aleatorias.
Muestra la clave solo una vez.
Guarda hash de la clave, no la clave en texto plano.
Permite rotación.
Permite revocación.
Asigna scopes.
Asigna expiración cuando aplique.
Registra uso.
No aceptes API keys en query string.
```

Evita:

```http
GET /api/data?api_key=secret
```

Mejor:

```http
Authorization: Bearer secret
```

O:

```http
X-API-Key: secret
```

---

## 40. Idempotencia

Para operaciones críticas, usa idempotency keys.

Ejemplo:

```http
POST /payments
Idempotency-Key: 5b6938c6-6c2e-4672-bf2f-7f9ef0d0f6ad
```

Sirve para evitar duplicados si el cliente reintenta por error de red.

Usos:

```text
Pagos.
Órdenes.
Reservas.
Creación de recursos costosos.
Webhooks.
```

---

# Parte VII: datos, criptografía y secretos

## 41. Clasificación de datos

Clasifica datos para aplicar controles adecuados.

```text
Público:
Contenido que puede publicarse.

Interno:
Información de operación no sensible.

Confidencial:
Datos de usuarios, contratos, métricas privadas.

Restringido:
Credenciales, tokens, datos financieros, salud, secretos comerciales.
```

Por cada clase define:

```text
Quién puede acceder.
Dónde se almacena.
Cuánto tiempo se retiene.
Cómo se cifra.
Cómo se audita.
Cómo se elimina.
```

---

## 42. Cifrado

Buenas prácticas:

```text
Usa TLS en tránsito.
Cifra backups.
Cifra discos o volúmenes.
Usa KMS o gestor de secretos.
No diseñes criptografía propia.
Usa librerías maduras.
Rota claves.
Separa claves de datos.
```

Regla:

```text
No inventes algoritmos criptográficos.
No implementes primitivas criptográficas desde cero.
```

---

## 43. Secretos

Secretos comunes:

```text
DATABASE_URL.
JWT_SECRET.
OAuth client secret.
API keys.
Private keys.
Webhook secrets.
Tokens de cloud.
Credenciales SMTP.
```

Buenas prácticas:

```text
No commitear secretos.
Usar variables de entorno o secret manager.
Escanear repositorios.
Rotar secretos filtrados.
Dar permisos mínimos.
Separar secretos por ambiente.
No imprimir secretos en logs.
```

Mala práctica:

```python
JWT_SECRET = "my-secret-in-code"
```

Mejor:

```python
JWT_SECRET = os.environ["JWT_SECRET"]
```

Pero en producción madura:

```text
Secret manager.
KMS.
IAM de workload.
Rotación.
Auditoría de acceso.
```

---

## 44. Logs seguros

Los logs son esenciales, pero pueden filtrar datos.

No loguees:

```text
Contraseñas.
Tokens.
Cookies.
Refresh tokens.
Números completos de tarjetas.
Documentos personales.
Headers Authorization.
Secretos.
```

Sí loguea:

```text
Timestamp.
Usuario o subject.
Tenant.
IP.
User agent.
Endpoint.
Método.
Status code.
Correlation ID.
Resultado de autorización.
Eventos críticos.
```

Ejemplo:

```json
{
    "event": "access_denied",
    "user_id": "user_123",
    "tenant_id": "tenant_456",
    "resource": "document",
    "resource_id": "doc_789",
    "status": 403,
    "correlation_id": "req_abc"
}
```

---

# Parte VIII: infraestructura y configuración

## 45. Headers de seguridad

Headers recomendados:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), camera=(), microphone=()
```

Notas:

```text
HSTS requiere HTTPS correcto.
CSP puede romper scripts si se activa sin pruebas.
Permissions-Policy reduce APIs disponibles al navegador.
```

---

## 46. TLS

Buenas prácticas:

```text
Usa HTTPS en todos los ambientes accesibles.
Redirige HTTP a HTTPS.
Habilita HSTS en producción.
Renueva certificados automáticamente.
Deshabilita protocolos y cifrados obsoletos.
```

---

## 47. Configuración segura

Revisa:

```text
DEBUG=false en producción.
Errores detallados desactivados.
Paneles admin protegidos.
Puertos internos no expuestos.
Bases de datos no públicas.
Buckets privados por defecto.
CORS restringido.
Credenciales separadas por ambiente.
Permisos cloud mínimos.
```

---

## 48. Contenedores

Buenas prácticas:

```text
Usa imágenes base oficiales y mínimas.
No ejecutes como root si no es necesario.
Escanea imágenes.
Fija versiones.
No incluyas secretos en la imagen.
Usa multi-stage builds.
Reduce paquetes instalados.
Define healthchecks.
Monta filesystem read-only cuando sea posible.
```

Dockerfile conceptual:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

RUN useradd -m appuser

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

USER appuser

CMD ["python", "-m", "app"]
```

---

## 49. CI/CD seguro

Buenas prácticas:

```text
Revisión de código obligatoria.
SAST.
Dependency scanning.
Secret scanning.
IaC scanning.
Builds reproducibles cuando sea posible.
Firmar artefactos si el riesgo lo exige.
Separar permisos de CI por repositorio.
No usar tokens permanentes amplios.
Aprobar despliegues a producción.
```

---

# Parte IX: técnicas típicas de ataque y mitigación

## 50. Tabla resumida

| Riesgo | Qué busca el atacante | Mitigación principal |
|---|---|---|
| SQL injection | Ejecutar SQL no autorizado | Queries parametrizadas, ORM, privilegios mínimos |
| XSS | Ejecutar JavaScript en navegador de víctima | Output encoding, sanitización, CSP |
| CSRF | Forzar requests autenticados | SameSite, CSRF token, Origin checks |
| IDOR/BOLA | Acceder a objetos ajenos | Autorización por recurso |
| SSRF | Hacer que el servidor llame redes internas | Allowlist, bloqueo IP privada, red aislada |
| Path traversal | Leer archivos fuera del directorio permitido | Normalizar rutas, usar IDs, validar path final |
| Credential stuffing | Probar credenciales filtradas | MFA, rate limit, detección de abuso |
| Clickjacking | Engañar clicks en iframe | CSP frame-ancestors |
| Supply chain | Comprometer dependencias/build | Lockfiles, scanning, revisión, SCA |
| Secrets leak | Usar claves filtradas | Secret scanning, rotación, secret manager |
| Insecure deserialization | Ejecutar objetos manipulados | Evitar deserialización peligrosa, firmar datos |
| Open redirect | Redirigir a sitios maliciosos | Allowlist de URLs destino |

---

## 51. SQL injection: revisión defensiva

Busca patrones como:

```text
f"SELECT ..."
"SELECT " + user_input
format() aplicado sobre SQL
ORDER BY dinámico sin allowlist
WHERE dinámico construido con strings
```

Checklist:

```text
[ ] Todos los valores usan parámetros.
[ ] Columnas dinámicas usan allowlist.
[ ] El usuario de DB no es superuser.
[ ] Errores SQL no se exponen al cliente.
[ ] Hay tests con entradas inesperadas.
```

---

## 52. XSS: revisión defensiva

Busca patrones como:

```text
innerHTML con datos de usuario.
dangerouslySetInnerHTML.
v-html en Vue.
HTML marcado como safe.
Markdown renderizado sin sanitizar.
URLs javascript:.
Templates sin autoescaping.
```

Checklist:

```text
[ ] Se usa textContent cuando corresponde.
[ ] Templates escapan por defecto.
[ ] HTML enriquecido se sanitiza.
[ ] CSP está configurada.
[ ] Cookies de sesión son HttpOnly.
[ ] No hay tokens sensibles en localStorage si la app puede evitarlo.
```

---

## 53. CSRF: revisión defensiva

Checklist:

```text
[ ] GET no cambia estado.
[ ] Cookies de sesión tienen SameSite.
[ ] Acciones críticas usan CSRF token u Origin checks.
[ ] CORS no permite orígenes arbitrarios con credenciales.
[ ] Acciones sensibles requieren reautenticación si aplica.
```

---

## 54. IDOR/BOLA: revisión defensiva

Checklist:

```text
[ ] Cada recurso verifica owner_id, tenant_id o relación.
[ ] Las consultas filtran por usuario/tenant.
[ ] No se confía en IDs ocultos del frontend.
[ ] Existen tests de acceso cruzado.
[ ] Admin y usuario normal se prueban por separado.
```

Ejemplo de test conceptual:

```python
def test_user_cannot_read_other_users_document(client, user_a_token, user_b_document):
    response = client.get(
        f"/documents/{user_b_document.id}",
        headers={"Authorization": f"Bearer {user_a_token}"},
    )

    assert response.status_code in {403, 404}
```

---

## 55. SSRF: revisión defensiva

Checklist:

```text
[ ] No se permiten URLs arbitrarias salvo necesidad clara.
[ ] Hay allowlist de dominios.
[ ] Se bloquean IPs privadas y metadata cloud.
[ ] Se revalida tras redirecciones.
[ ] Hay timeout y límite de tamaño.
[ ] El servicio que descarga no tiene acceso a redes internas sensibles.
```

---

# Parte X: auditorías de seguridad

## 56. Tipos de auditoría

```text
Revisión de arquitectura.
Threat modeling.
Code review seguro.
SAST.
DAST.
SCA o dependency scanning.
Secret scanning.
Revisión de infraestructura.
Revisión de cloud IAM.
Pentest.
Revisión de logs y monitoreo.
Revisión de cumplimiento.
```

Ninguna técnica cubre todo. Combínalas.

---

## 57. Auditoría paso a paso

### Paso 1: definir alcance

```text
Aplicaciones.
APIs.
Repositorios.
Ambientes.
Dominios.
Infraestructura.
Terceros.
Datos sensibles.
Fechas.
Restricciones.
```

Resultado esperado:

```text
Documento de alcance.
Responsables.
Ventanas de prueba.
Contactos de emergencia.
Reglas de engagement.
```

---

### Paso 2: inventario de activos

Registra:

```text
Dominios y subdominios.
APIs.
Repositorios.
Bases de datos.
Buckets.
Colas.
Jobs.
Servicios externos.
Cuentas cloud.
Secretos.
Roles.
Paneles admin.
```

---

### Paso 3: clasificación de datos

Identifica:

```text
Datos personales.
Datos financieros.
Datos de salud.
Credenciales.
Tokens.
Archivos privados.
Logs sensibles.
Datos regulados.
```

Define:

```text
Retención.
Cifrado.
Acceso.
Auditoría.
Borrado.
```

---

### Paso 4: threat modeling

Método simple:

```text
1. Dibuja arquitectura.
2. Marca límites de confianza.
3. Identifica entradas externas.
4. Identifica datos sensibles.
5. Lista amenazas por componente.
6. Define mitigaciones.
7. Asigna responsables.
```

Preguntas:

```text
¿Qué controla el atacante?
¿Qué pasa si este servicio cae?
¿Qué pasa si este token se filtra?
¿Qué pasa si este usuario cambia el tenant_id?
¿Qué pasa si el proveedor externo responde datos maliciosos?
```

---

### Paso 5: revisión de autenticación

Checklist:

```text
[ ] HTTPS obligatorio.
[ ] Contraseñas hasheadas con algoritmo adecuado.
[ ] MFA disponible o requerido según riesgo.
[ ] Rate limit en login.
[ ] Recuperación de contraseña segura.
[ ] Tokens con expiración.
[ ] Logout invalida sesión o token cuando corresponde.
[ ] Cookies seguras.
[ ] No hay tokens en URLs.
[ ] No hay credenciales en logs.
```

---

### Paso 6: revisión de autorización

Checklist:

```text
[ ] Permisos definidos formalmente.
[ ] RBAC/ABAC/ReBAC documentado.
[ ] Backend verifica permisos en cada endpoint.
[ ] Consultas filtran por owner/tenant.
[ ] Admin endpoints están protegidos.
[ ] Tests cubren acceso cruzado.
[ ] No hay endpoints "temporales" sin control.
```

---

### Paso 7: revisión de entrada y salida

Checklist:

```text
[ ] Schemas para body, query, path y headers.
[ ] Límites de tamaño.
[ ] Allowlists para valores cerrados.
[ ] Sanitización donde se renderiza HTML.
[ ] Output encoding según contexto.
[ ] Archivos validados.
[ ] Webhooks firmados.
```

---

### Paso 8: revisión de base de datos

Checklist:

```text
[ ] Queries parametrizadas.
[ ] ORM usado correctamente.
[ ] Usuario de aplicación con permisos mínimos.
[ ] Migraciones revisadas.
[ ] Backups cifrados.
[ ] Datos sensibles cifrados o tokenizados si aplica.
[ ] Acceso externo bloqueado.
[ ] Logs SQL no exponen datos sensibles.
```

---

### Paso 9: revisión de frontend

Checklist:

```text
[ ] No hay tokens sensibles innecesarios en localStorage.
[ ] No hay innerHTML con datos no confiables.
[ ] CSP configurada.
[ ] Dependencias auditadas.
[ ] Formularios no confían en validación cliente.
[ ] CORS está restringido.
[ ] Headers de seguridad activos.
```

---

### Paso 10: revisión de infraestructura

Checklist:

```text
[ ] DEBUG desactivado.
[ ] TLS correcto.
[ ] HSTS en producción.
[ ] Puertos internos cerrados.
[ ] Buckets privados.
[ ] IAM mínimo.
[ ] Contenedores no root.
[ ] Imágenes escaneadas.
[ ] Secretos fuera del repositorio.
[ ] Backups probados.
```

---

### Paso 11: pruebas automatizadas

Integra:

```text
SAST:
Busca patrones inseguros en código.

DAST:
Prueba la app desplegada desde fuera.

SCA:
Analiza vulnerabilidades en dependencias.

Secret scanning:
Detecta claves filtradas.

IaC scanning:
Revisa Terraform, Kubernetes, Docker, etc.

Container scanning:
Busca vulnerabilidades en imágenes.
```

---

### Paso 12: reporte

Un buen hallazgo debe incluir:

```text
Título.
Severidad.
Activo afectado.
Descripción.
Impacto.
Evidencia.
Pasos de reproducción seguros.
Recomendación.
Responsable.
Fecha objetivo.
Estado.
```

Severidad recomendada:

```text
Crítica.
Alta.
Media.
Baja.
Informativa.
```

No todos los hallazgos con CVSS alto son críticos para tu contexto. Ajusta por exposición, datos afectados y facilidad de explotación.

---

## 58. Plantilla de hallazgo

```markdown
## Hallazgo: Acceso a documentos de otros usuarios

Severidad: Alta

### Descripción

El endpoint `GET /documents/{document_id}` permite acceder a documentos de otros usuarios al modificar el identificador del documento.

### Impacto

Un usuario autenticado podría leer información privada de otros usuarios.

### Evidencia

Usuario A pudo solicitar el documento `doc_123`, perteneciente a Usuario B, y recibió respuesta `200 OK`.

### Recomendación

Agregar validación de autorización por recurso:

- Verificar `owner_id`.
- Verificar `tenant_id`.
- Retornar `403` o `404`.
- Agregar tests automatizados de acceso cruzado.

### Estado

Pendiente.
```

---

## 59. Priorización de remediación

Prioriza por:

```text
Exposición pública.
Datos sensibles.
Facilidad de explotación.
Impacto legal o financiero.
Existencia de explotación activa.
Número de usuarios afectados.
Privilegios requeridos.
Compensating controls existentes.
```

Orden típico:

```text
1. Exposición de secretos.
2. Acceso no autorizado a datos.
3. RCE o command injection.
4. SQL injection explotable.
5. Fallas críticas de autenticación.
6. SSRF hacia metadata cloud.
7. XSS almacenado en zonas sensibles.
8. Dependencias vulnerables explotables.
```

---

# Parte XI: operación segura

## 60. Monitoreo

Monitorea:

```text
Logins fallidos.
Cambios de contraseña.
Cambios de MFA.
Creación de API keys.
Uso de API keys.
Accesos 403 repetidos.
Errores 500.
Picos de tráfico.
Rate limits alcanzados.
Subidas de archivos.
Cambios de roles.
Exportaciones masivas.
Cambios de configuración.
```

Alertas útiles:

```text
Muchos intentos de login contra una cuenta.
Muchos intentos de login desde una IP.
Usuario normal intentando endpoints admin.
Descarga masiva de datos.
Creación sospechosa de tokens.
Desactivación de MFA.
Errores 500 después de despliegue.
```

---

## 61. Respuesta a incidentes

Define antes del incidente:

```text
Quién coordina.
Cómo se reporta.
Cómo se aísla.
Cómo se revocan secretos.
Cómo se preserva evidencia.
Cómo se comunica.
Cómo se restaura.
Cómo se documenta.
```

Fases:

```text
Preparación.
Detección.
Contención.
Erradicación.
Recuperación.
Postmortem.
```

Checklist inmediato si se filtra un secreto:

```text
[ ] Revocar secreto.
[ ] Crear secreto nuevo.
[ ] Buscar uso indebido.
[ ] Revisar logs.
[ ] Identificar alcance.
[ ] Rotar secretos relacionados.
[ ] Agregar detección para evitar repetición.
```

---

## 62. Backups y recuperación

Buenas prácticas:

```text
Backups automáticos.
Cifrado de backups.
Retención definida.
Pruebas de restauración.
Separación de permisos.
Backups fuera del entorno principal.
Protección contra borrado accidental.
```

Regla:

```text
Un backup no probado no es garantía.
```

---

## 63. Seguridad en ambientes

Ambientes:

```text
Local.
Desarrollo.
Staging.
Producción.
```

Reglas:

```text
No uses datos productivos reales en desarrollo salvo anonimización.
No compartas secretos entre ambientes.
No uses permisos de producción para pruebas.
No expongas staging públicamente sin protección.
No dejes usuarios demo con claves débiles.
```

---

# Parte XII: checklist final

## 64. Checklist para desarrollo web seguro

```text
Autenticación
[ ] HTTPS obligatorio.
[ ] Passwords hasheadas con Argon2id/bcrypt/scrypt.
[ ] MFA para cuentas sensibles.
[ ] Rate limit en login.
[ ] Recuperación de contraseña segura.
[ ] Tokens no aparecen en URLs.

Sesiones y cookies
[ ] Cookies con HttpOnly.
[ ] Cookies con Secure.
[ ] Cookies con SameSite.
[ ] Expiración definida.
[ ] Logout invalida sesión.
[ ] Sesión se regenera en login.

Autorización
[ ] Backend verifica permisos.
[ ] Se valida owner_id/tenant_id.
[ ] No hay IDOR/BOLA.
[ ] Admin endpoints protegidos.
[ ] Tests cubren acceso cruzado.

Backend
[ ] Validación de entrada.
[ ] Queries parametrizadas.
[ ] Errores internos ocultos.
[ ] Rate limiting.
[ ] Webhooks firmados.
[ ] File uploads controlados.
[ ] Secretos fuera del código.

Frontend
[ ] No se usa innerHTML con datos no confiables.
[ ] HTML enriquecido se sanitiza.
[ ] CSP configurada.
[ ] Tokens sensibles no están en localStorage si puede evitarse.
[ ] CORS restringido.
[ ] Dependencias auditadas.

Infraestructura
[ ] DEBUG apagado.
[ ] HSTS en producción.
[ ] Buckets privados.
[ ] DB no pública.
[ ] IAM mínimo.
[ ] Contenedores no root.
[ ] Imágenes escaneadas.

Auditoría
[ ] SAST.
[ ] DAST.
[ ] SCA.
[ ] Secret scanning.
[ ] IaC scanning.
[ ] Threat model.
[ ] Revisión manual de autorización.
[ ] Reporte con severidad y remediación.

Operación
[ ] Logs seguros.
[ ] Alertas críticas.
[ ] Backups cifrados y probados.
[ ] Plan de respuesta a incidentes.
[ ] Rotación de secretos.
```

---

# Parte XIII: base mínima recomendada para una API web

## 65. Política mínima

Para una API privada con frontend web:

```text
Transporte:
- HTTPS.
- HSTS en producción.

Autenticación:
- Cookie de sesión HttpOnly, Secure, SameSite=Lax o Strict.
- MFA para administradores.
- Rate limiting en login.
- Password hashing con Argon2id/bcrypt/scrypt.

Autorización:
- Verificación por recurso.
- tenant_id/owner_id en consultas.
- Roles y permisos documentados.

Backend:
- Validación estricta.
- Queries parametrizadas.
- Errores genéricos.
- Logs con correlation_id.
- Secrets en secret manager.

Frontend:
- No guardar access tokens sensibles en localStorage si puede evitarse.
- CSP.
- Sanitización de HTML.
- CORS con allowlist.

CI/CD:
- Secret scanning.
- Dependency scanning.
- SAST.
- Revisión de código.

Operación:
- Backups probados.
- Alertas de login y acceso denegado.
- Plan de respuesta a incidentes.
```

---

# Fuentes de referencia recomendadas

```text
- OWASP Top 10.
- OWASP Application Security Verification Standard, ASVS.
- OWASP Cheat Sheet Series.
- OWASP SQL Injection Prevention Cheat Sheet.
- OWASP Cross-Site Scripting Prevention Cheat Sheet.
- OWASP Session Management Cheat Sheet.
- IETF RFC 9700: Best Current Practice for OAuth 2.0 Security.
- NIST SP 800-63B: Digital Identity Guidelines.
- MDN Web Docs: Set-Cookie.
- MDN Web Docs: HTTP cookies.
- MDN Web Docs: Content Security Policy.
- CISA Secure by Design.
- CIS Benchmarks.
```
