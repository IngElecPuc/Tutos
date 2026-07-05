# Guía práctica de PEP 8 para Python

## Objetivo

Esta guía explica cómo aplicar PEP 8 en proyectos Python reales. Está orientada a escribir código legible, consistente y mantenible. Incluye reglas de estilo, nombres, imports, espacios, líneas, comentarios, docstrings, type hints, organización de archivos y configuración de herramientas modernas como `ruff`, `black`, `isort`, `flake8` y `pycodestyle`.

Está organizada en tres niveles:

```text
1. Básico: formato, nombres, imports, espacios, líneas y estructura general.
2. Intermedio: funciones, clases, excepciones, comentarios, docstrings, type hints y organización de módulos.
3. Avanzado: configuración de linters/formatters, CI/CD, pre-commit, migración de código legado, excepciones justificadas y estándares de equipo.
```

PEP 8 no debe entenderse como una ley mecánica. Su objetivo principal es mejorar la legibilidad. La propia guía PEP 8 indica que la consistencia importa, pero también que hay casos donde romper una regla puede ser razonable si mejora la claridad o mantiene coherencia local.

---

# Parte I: fundamentos

## 1. Qué es PEP 8

PEP 8 es la guía de estilo para código Python. Define convenciones sobre:

```text
Indentación.
Longitud de línea.
Imports.
Espacios.
Saltos de línea.
Nombres.
Comentarios.
Docstrings.
Comparaciones.
Excepciones.
Diseño general de código.
```

Su propósito es que el código Python sea:

```text
Legible.
Consistente.
Fácil de revisar.
Fácil de mantener.
Familiar para otros desarrolladores Python.
```

Ejemplo de código poco consistente:

```python
def getUserName( user_id ):
  if(user_id==1): return "Ana"
  else: return "Unknown"
```

Ejemplo más alineado con PEP 8:

```python
def get_user_name(user_id: int) -> str:
    if user_id == 1:
        return "Ana"

    return "Unknown"
```

---

## 2. Principio central: legibilidad

PEP 8 existe porque el código se lee más veces de las que se escribe.

Prioridad práctica:

```text
1. Código correcto.
2. Código claro.
3. Código consistente.
4. Código fácil de cambiar.
5. Código corto, solo si no pierde claridad.
```

No escribas código críptico solo para hacerlo más breve.

Malo:

```python
r = [x for x in xs if x.a and not x.b and x.c > 3]
```

Mejor:

```python
active_items = [
    item
    for item in items
    if item.is_active and not item.is_deleted and item.priority > 3
]
```

---

## 3. Consistencia

Regla práctica:

```text
Primero sé consistente con el proyecto.
Luego con el módulo.
Luego con PEP 8.
```

Si estás modificando un archivo existente que usa un estilo ligeramente distinto, no mezcles estilos sin razón. Idealmente, aplica un formateador al archivo completo si el equipo lo acepta.

---

# Parte II: formato básico

## 4. Indentación

Usa 4 espacios por nivel de indentación.

Correcto:

```python
def process_order(order: Order) -> None:
    if order.is_paid:
        send_confirmation(order)
```

Incorrecto:

```python
def process_order(order):
  if order.is_paid:
      send_confirmation(order)
```

Reglas:

```text
No mezcles tabs y espacios.
Usa 4 espacios.
Configura el editor para insertar espacios.
```

---

## 5. Indentación en llamadas largas

Cuando una llamada no cabe en una línea, usa saltos claros.

Correcto:

```python
create_user(
    email="user@example.com",
    first_name="Ana",
    last_name="Pérez",
    is_active=True,
)
```

También válido:

```python
result = calculate_total(
    subtotal,
    tax_rate,
    discount,
)
```

Evita:

```python
result = calculate_total(subtotal,
                         tax_rate,
                         discount)
```

Aunque es válido, suele ser más frágil ante renombres y cambios de longitud.

---

## 6. Longitud de línea

PEP 8 recomienda 79 caracteres para código y 72 para comentarios/docstrings. En proyectos modernos, muchos equipos usan 88 o 100/120 caracteres. `black` usa 88 por defecto.

Recomendación práctica:

```text
Código de librería pública o estilo estricto: 79.
Proyectos modernos con black/ruff format: 88.
Equipos internos con pantallas amplias: 100 o 120, si está documentado.
```

Lo importante es que el equipo defina un valor y lo automatice.

Ejemplo de línea larga:

```python
response = client.create_user_with_profile_and_permissions(email="user@example.com", role="admin", tenant_id="tenant_123")
```

Mejor:

```python
response = client.create_user_with_profile_and_permissions(
    email="user@example.com",
    role="admin",
    tenant_id="tenant_123",
)
```

---

## 7. Saltos de línea antes de operadores

PEP 8 prefiere romper antes del operador binario.

Correcto:

```python
total = (
    subtotal
    + tax
    - discount
    + shipping_cost
)
```

Menos recomendado:

```python
total = (
    subtotal +
    tax -
    discount +
    shipping_cost
)
```

La primera forma permite leer operadores al inicio de cada línea.

---

## 8. Líneas en blanco

Usa líneas en blanco para separar unidades lógicas.

Reglas comunes:

```text
Dos líneas en blanco entre funciones o clases de nivel superior.
Una línea en blanco entre métodos dentro de una clase.
Líneas en blanco internas para separar bloques lógicos.
```

Ejemplo:

```python
class UserService:
    def create_user(self, email: str) -> User:
        normalized_email = normalize_email(email)

        user = User(email=normalized_email)
        self.repository.save(user)

        return user

    def delete_user(self, user_id: str) -> None:
        self.repository.delete(user_id)


def normalize_email(email: str) -> str:
    return email.strip().lower()
```

---

## 9. Imports

Los imports van al inicio del archivo, después del docstring del módulo y antes de constantes o código.

Orden recomendado:

```text
1. Librería estándar.
2. Dependencias de terceros.
3. Imports locales del proyecto.
```

Ejemplo:

```python
from pathlib import Path
from typing import Any

import requests
from pydantic import BaseModel

from my_app.config import settings
from my_app.users.models import User
```

Separa cada grupo con una línea en blanco.

---

## 10. Un import por línea

Correcto:

```python
import os
import sys
```

Incorrecto:

```python
import os, sys
```

Excepción razonable:

```python
from subprocess import PIPE, Popen
```

Aun así, si la lista crece, usa múltiples líneas:

```python
from subprocess import (
    PIPE,
    Popen,
    TimeoutExpired,
)
```

---

## 11. Imports absolutos vs relativos

PEP 8 recomienda imports absolutos porque son más claros.

Preferido:

```python
from my_app.users.services import UserService
```

Relativo aceptable en paquetes:

```python
from .services import UserService
from ..config import settings
```

Evita relativos demasiado profundos:

```python
from ....shared.utils import normalize_email
```

Si necesitas muchos `....`, probablemente la estructura del paquete necesita revisión.

---

## 12. Wildcard imports

Evita:

```python
from module import *
```

Problemas:

```text
Oculta de dónde viene cada nombre.
Dificulta análisis estático.
Puede pisar nombres.
Complica mantenimiento.
```

Usa:

```python
from module import SomeClass, some_function
```

Excepción limitada:

```text
Módulos diseñados explícitamente para reexportar una API pública.
```

---

# Parte III: espacios

## 13. Espacios alrededor de operadores

Correcto:

```python
total = subtotal + tax - discount
is_valid = age >= 18 and country == "CL"
```

Incorrecto:

```python
total=subtotal+tax-discount
is_valid=age>=18 and country=="CL"
```

Operadores que llevan espacios:

```text
=
+
-
*
/
==
!=
<
>
<=
>=
and
or
in
is
```

---

## 14. No usar espacios innecesarios

Incorrecto:

```python
spam( ham[ 1 ], { eggs: 2 } )
```

Correcto:

```python
spam(ham[1], {eggs: 2})
```

Incorrecto:

```python
foo = (0, )
```

Correcto:

```python
foo = (0,)
```

---

## 15. Espacios en argumentos con valor por defecto

PEP 8 distingue entre argumentos sin anotaciones y con anotaciones.

Sin type hints:

```python
def connect(host="localhost", port=5432):
    ...
```

Con type hints:

```python
def connect(host: str = "localhost", port: int = 5432) -> None:
    ...
```

Incorrecto:

```python
def connect(host: str="localhost", port: int=5432) -> None:
    ...
```

---

## 16. Espacios en slicing

PEP 8 trata `:` en slicing como operador de baja prioridad. En casos simples, sin espacios.

Correcto:

```python
items[1:3]
items[:10]
items[2:]
items[::2]
```

Casos más complejos:

```python
items[start : stop + offset]
items[start + offset : stop]
```

Mantén simetría:

```python
items[start:stop]
items[start : stop + 1]
```

Evita mezclar de forma arbitraria.

---

# Parte IV: nombres

## 17. Principios de naming

Los nombres deben comunicar intención.

Malo:

```python
d = get_data()
x = calculate(d)
```

Mejor:

```python
orders = get_orders()
total_revenue = calculate_revenue(orders)
```

Reglas:

```text
Nombres descriptivos.
Evitar abreviaturas oscuras.
Evitar nombres de una letra salvo contadores simples.
Usar convenciones según tipo de entidad.
```

---

## 18. Convenciones principales

| Elemento | Convención | Ejemplo |
|---|---|---|
| Módulos | `lowercase_with_underscores` | `user_service.py` |
| Paquetes | `lowercase` | `users` |
| Funciones | `snake_case` | `create_user()` |
| Variables | `snake_case` | `user_email` |
| Métodos | `snake_case` | `get_total()` |
| Clases | `CapWords` / PascalCase | `UserService` |
| Excepciones | `CapWords` con sufijo Error | `ValidationError` |
| Constantes | `UPPER_CASE` | `MAX_RETRIES` |
| Type variables | nombres cortos o descriptivos | `T`, `UserT` |
| Parámetro self | `self` | `self.email` |
| Parámetro cls | `cls` | `cls.from_json()` |

---

## 19. Funciones y variables

Correcto:

```python
def calculate_invoice_total(invoice_items: list[InvoiceItem]) -> Decimal:
    ...
```

Incorrecto:

```python
def CalculateInvoiceTotal(invoiceItems):
    ...
```

Evita nombres demasiado genéricos:

```python
data = get_data()
result = process(data)
```

Mejor:

```python
raw_orders = get_raw_orders()
normalized_orders = normalize_orders(raw_orders)
```

---

## 20. Clases

Las clases usan `CapWords`.

Correcto:

```python
class UserRepository:
    pass


class PaymentGateway:
    pass
```

Incorrecto:

```python
class user_repository:
    pass
```

Las excepciones deberían terminar en `Error`:

```python
class PaymentDeclinedError(Exception):
    pass
```

---

## 21. Constantes

Las constantes de módulo usan mayúsculas.

```python
MAX_RETRIES = 3
DEFAULT_TIMEOUT_SECONDS = 30
SUPPORTED_FORMATS = {"csv", "json", "parquet"}
```

En Python no existen constantes reales. La convención indica que no deberían modificarse.

---

## 22. Nombres privados

Un guion bajo inicial indica uso interno.

```python
def _normalize_email(email: str) -> str:
    return email.strip().lower()
```

Atributo interno:

```python
class UserService:
    def __init__(self) -> None:
        self._cache: dict[str, User] = {}
```

No es seguridad. Es convención.

---

## 23. Doble guion bajo

El doble guion bajo activa name mangling en clases.

```python
class Account:
    def __init__(self) -> None:
        self.__balance = 0
```

Uso recomendado:

```text
Raro.
Solo cuando necesitas evitar colisiones en herencia.
```

No lo uses como “privado fuerte” por costumbre. Normalmente `_balance` basta.

---

## 24. Evitar conflictos con palabras reservadas

Si necesitas un nombre que choca con palabra reservada, agrega `_` al final.

```python
class_ = "premium"
from_ = "2026-01-01"
```

También aplica a built-ins si hay riesgo de confusión:

```python
list_ = [1, 2, 3]
id_ = "user_123"
```

Evita sobrescribir built-ins:

```python
list = [1, 2, 3]  # Malo
```

---

## 25. Nombres booleanos

Usa nombres que respondan sí/no.

Correcto:

```python
is_active = True
has_permission = False
can_delete = True
should_retry = False
```

Menos claro:

```python
active = True
permission = False
delete = True
retry = False
```

Funciones booleanas:

```python
def is_valid_email(email: str) -> bool:
    ...


def has_access(user: User, resource: Resource) -> bool:
    ...
```

---

# Parte V: funciones

## 26. Funciones pequeñas y claras

Una función debería hacer una cosa principal.

Malo:

```python
def process_user(payload):
    # validate
    # normalize
    # save
    # send email
    # log audit
    ...
```

Mejor:

```python
def create_user(payload: dict) -> User:
    user_input = validate_user_payload(payload)
    user = build_user(user_input)
    save_user(user)
    send_welcome_email(user)
    log_user_created(user)

    return user
```

No se trata de dividir por dividir. Se trata de separar responsabilidades que cambian por motivos distintos.

---

## 27. Argumentos

Evita funciones con demasiados argumentos posicionales.

Difícil de leer:

```python
create_user("Ana", "Pérez", "ana@example.com", True, False, "admin")
```

Mejor:

```python
create_user(
    first_name="Ana",
    last_name="Pérez",
    email="ana@example.com",
    is_active=True,
    is_verified=False,
    role="admin",
)
```

Si hay muchos campos, usa un objeto de entrada:

```python
@dataclass(frozen=True)
class CreateUserInput:
    first_name: str
    last_name: str
    email: str
    role: str
    is_active: bool = True


def create_user(user_input: CreateUserInput) -> User:
    ...
```

---

## 28. Keyword-only arguments

Útiles para evitar errores con booleanos o argumentos ambiguos.

```python
def send_email(
    to: str,
    subject: str,
    *,
    send_copy: bool = False,
    urgent: bool = False,
) -> None:
    ...
```

Uso:

```python
send_email(
    "user@example.com",
    "Welcome",
    send_copy=True,
    urgent=False,
)
```

Esto evita:

```python
send_email("user@example.com", "Welcome", True, False)
```

---

## 29. Retornos explícitos

Sé consistente en retornos.

Malo:

```python
def find_user(user_id: str):
    if user_id:
        return get_user(user_id)
```

Aquí a veces retorna `User`, a veces `None` implícito.

Mejor:

```python
def find_user(user_id: str) -> User | None:
    if not user_id:
        return None

    return get_user(user_id)
```

---

## 30. Comparar con None

Correcto:

```python
if user is None:
    ...
```

Incorrecto:

```python
if user == None:
    ...
```

Para no None:

```python
if user is not None:
    ...
```

---

## 31. Comparar booleanos

Correcto:

```python
if is_active:
    ...
```

Incorrecto:

```python
if is_active == True:
    ...
```

Para falso:

```python
if not is_active:
    ...
```

---

## 32. No comparar con `len(...) == 0`

Preferido:

```python
if not items:
    ...
```

En vez de:

```python
if len(items) == 0:
    ...
```

Para no vacío:

```python
if items:
    ...
```

Esto aplica a listas, tuplas, sets, dicts y strings.

---

# Parte VI: clases

## 33. Diseño de clases

Una clase debería representar una entidad, servicio, política o abstracción clara.

Ejemplo razonable:

```python
class InvoiceCalculator:
    def calculate_total(self, invoice: Invoice) -> Decimal:
        ...
```

Evita clases sin estado ni abstracción real:

```python
class MathUtils:
    @staticmethod
    def add(a, b):
        return a + b
```

En Python, una función de módulo suele ser mejor:

```python
def add(a: int, b: int) -> int:
    return a + b
```

---

## 34. Orden interno de una clase

Orden recomendado:

```text
1. Docstring.
2. Atributos de clase.
3. __init__.
4. Métodos públicos.
5. Métodos protegidos/internos.
6. Métodos especiales, si el equipo lo prefiere agrupado.
```

Ejemplo:

```python
class UserService:
    """Service for user operations."""

    default_role = "user"

    def __init__(self, repository: UserRepository) -> None:
        self._repository = repository

    def create_user(self, email: str) -> User:
        normalized_email = self._normalize_email(email)
        return self._repository.create(email=normalized_email)

    def _normalize_email(self, email: str) -> str:
        return email.strip().lower()
```

---

## 35. Métodos estáticos y de clase

Usa `@staticmethod` cuando el método no necesita `self` ni `cls`.

```python
class EmailValidator:
    @staticmethod
    def is_valid(email: str) -> bool:
        return "@" in email
```

Pero una función puede ser mejor:

```python
def is_valid_email(email: str) -> bool:
    return "@" in email
```

Usa `@classmethod` para constructores alternativos.

```python
@dataclass
class User:
    email: str
    name: str

    @classmethod
    def from_dict(cls, data: dict[str, str]) -> "User":
        return cls(
            email=data["email"],
            name=data["name"],
        )
```

---

## 36. Dataclasses

Para objetos de datos simples, usa `dataclass`.

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Product:
    id: str
    name: str
    price: Decimal
```

Ventajas:

```text
Menos boilerplate.
Representación clara.
Comparación automática.
Opcionalmente inmutable con frozen=True.
```

No uses dataclass para servicios con mucha lógica de infraestructura.

---

# Parte VII: comentarios y docstrings

## 37. Comentarios

Un comentario debe explicar por qué, no repetir qué hace el código.

Malo:

```python
# Add one to counter
counter += 1
```

Mejor:

```python
# Retry count starts at 1 because the first attempt already failed.
retry_count += 1
```

Reglas:

```text
Mantén comentarios actualizados.
Elimina comentarios obsoletos.
No comentes código muerto.
No uses comentarios para justificar código confuso si puedes mejorar el código.
```

---

## 38. Comentarios inline

Úsalos con moderación.

Correcto:

```python
timeout = 60  # External provider can take up to 45 seconds.
```

Evita:

```python
timeout = 60 #timeout
```

PEP 8 recomienda separar comentario inline con al menos dos espacios.

---

## 39. Docstrings

Docstrings documentan módulos, clases, funciones y métodos públicos.

Ejemplo:

```python
def calculate_total(items: list[InvoiceItem]) -> Decimal:
    """Calculate invoice total including taxes and discounts."""
    ...
```

Para funciones complejas:

```python
def retry_payment(payment_id: str, *, max_attempts: int = 3) -> PaymentResult:
    """Retry a failed payment.

    Args:
        payment_id: Identifier of the payment to retry.
        max_attempts: Maximum number of retry attempts.

    Returns:
        Final payment result.

    Raises:
        PaymentNotFoundError: If the payment does not exist.
        PaymentAlreadyCapturedError: If the payment was already captured.
    """
    ...
```

---

## 40. PEP 257

PEP 257 complementa PEP 8 con convenciones de docstrings.

Reglas prácticas:

```text
Usa triple comillas dobles.
La primera línea debe ser un resumen.
La primera línea debe terminar con punto.
En docstrings largos, deja una línea en blanco después del resumen.
Documenta API pública.
No documentes obviedades.
```

Correcto:

```python
def normalize_email(email: str) -> str:
    """Normalize an email address."""
    return email.strip().lower()
```

---

# Parte VIII: excepciones

## 41. Captura excepciones específicas

Malo:

```python
try:
    process_file(path)
except Exception:
    pass
```

Mejor:

```python
try:
    process_file(path)
except FileNotFoundError:
    logger.warning("File not found", extra={"path": str(path)})
```

Regla:

```text
Captura lo que puedes manejar.
Deja propagar lo que no puedes manejar.
```

---

## 42. No silenciar errores

Malo:

```python
try:
    send_email(user)
except Exception:
    pass
```

Mejor:

```python
try:
    send_email(user)
except EmailProviderError:
    logger.exception(
        "Could not send email",
        extra={"user_id": user.id},
    )
    raise
```

Si decides no relanzar, debe haber una razón clara.

---

## 43. Encadenar excepciones

Usa `raise ... from ...` para mantener causa.

```python
try:
    payload = json.loads(raw_payload)
except json.JSONDecodeError as exc:
    raise InvalidPayloadError("Payload is not valid JSON") from exc
```

Evita perder contexto.

---

## 44. Definir excepciones propias

```python
class AppError(Exception):
    """Base application error."""


class UserNotFoundError(AppError):
    """Raised when a user does not exist."""


class PermissionDeniedError(AppError):
    """Raised when a user cannot access a resource."""
```

Buenas prácticas:

```text
Sufijo Error.
Jerarquía simple.
No crear excepciones innecesarias.
Usar excepciones propias en bordes de dominio.
```

---

# Parte IX: type hints y PEP 8

## 45. Type hints

PEP 8 no es una guía completa de typing, pero el estilo moderno de Python usa anotaciones.

Ejemplo:

```python
def get_user(user_id: str) -> User | None:
    ...
```

Variables cuando ayudan:

```python
users_by_id: dict[str, User] = {}
```

No anotes lo obvio si no aporta:

```python
count: int = 0
```

Puede estar bien en contextos de inferencia difícil, pero no es necesario por defecto.

---

## 46. Imports para typing

En Python moderno, puedes usar tipos built-in:

```python
def process(items: list[str]) -> dict[str, int]:
    ...
```

En versiones antiguas se usaba:

```python
from typing import Dict, List
```

Para anotaciones complejas:

```python
from collections.abc import Callable, Iterable, Mapping, Sequence
```

Ejemplo:

```python
def filter_items(
    items: Iterable[Item],
    predicate: Callable[[Item], bool],
) -> list[Item]:
    return [item for item in items if predicate(item)]
```

---

## 47. `Any`

Evita abusar de `Any`.

Malo:

```python
def process(payload: Any) -> Any:
    ...
```

Mejor:

```python
def process(payload: Mapping[str, object]) -> ProcessResult:
    ...
```

Usa `Any` cuando:

```text
Interactúas con código dinámico.
No hay forma razonable de expresar el tipo.
Estás migrando gradualmente.
```

---

## 48. `Optional`

En Python 3.10+:

```python
def find_user(user_id: str) -> User | None:
    ...
```

Equivalente antiguo:

```python
from typing import Optional

def find_user(user_id: str) -> Optional[User]:
    ...
```

Regla:

```text
Si puede retornar None, decláralo.
```

---

## 49. Protocolos

Para duck typing tipado:

```python
from typing import Protocol


class EmailSender(Protocol):
    def send(self, to: str, subject: str, body: str) -> None:
        ...


def notify_user(sender: EmailSender, email: str) -> None:
    sender.send(
        to=email,
        subject="Welcome",
        body="Thanks for joining.",
    )
```

Esto evita depender de una clase base cuando solo importa la interfaz.

---

# Parte X: expresiones, comprehensions y legibilidad

## 50. List comprehensions

Buenas para transformaciones simples.

```python
names = [user.name for user in users]
```

Con filtro:

```python
active_users = [user for user in users if user.is_active]
```

Evita comprehensions demasiado complejas:

```python
result = [
    transform(x)
    for group in groups
    for x in group.items
    if x.is_valid and x.score > threshold and not x.deleted
]
```

Si se vuelve difícil de leer, usa un loop.

---

## 51. Generators

Usa generator expressions para evaluación perezosa.

```python
total = sum(item.price for item in items)
```

Para funciones que producen muchos elementos:

```python
def iter_active_users(users: Iterable[User]) -> Iterator[User]:
    for user in users:
        if user.is_active:
            yield user
```

---

## 52. Ternarios

Úsalos solo si son simples.

Correcto:

```python
label = "active" if user.is_active else "inactive"
```

Evita:

```python
result = "A" if x > 10 else "B" if x > 5 else "C"
```

Mejor:

```python
if score > 10:
    result = "A"
elif score > 5:
    result = "B"
else:
    result = "C"
```

---

## 53. Asignación walrus

El operador `:=` puede ser útil, pero no debe reducir claridad.

Correcto:

```python
if match := pattern.search(text):
    return match.group(1)
```

Evita:

```python
while chunk := file.read(size) if ready else None:
    ...
```

Regla:

```text
Usa := cuando elimina repetición sin ocultar lógica.
```

---

# Parte XI: módulos y paquetes

## 54. Tamaño de módulos

Un módulo debe tener una responsabilidad clara.

Ejemplo:

```text
users/
    __init__.py
    models.py
    schemas.py
    services.py
    repository.py
    exceptions.py
```

Evita:

```text
utils.py gigante con 200 funciones sin relación.
```

Mejor:

```text
date_utils.py
email_utils.py
string_utils.py
```

O, mejor aún, módulos de dominio:

```text
billing/
    invoices.py
    taxes.py
    payments.py
```

---

## 55. Código ejecutable

Evita ejecutar lógica pesada al importar un módulo.

Malo:

```python
# report.py
data = fetch_large_dataset()
generate_report(data)
```

Mejor:

```python
def main() -> None:
    data = fetch_large_dataset()
    generate_report(data)


if __name__ == "__main__":
    main()
```

---

## 56. `__all__`

Útil para definir API pública de un módulo.

```python
__all__ = [
    "User",
    "UserService",
    "UserNotFoundError",
]
```

No es obligatorio en todos los módulos.

Úsalo cuando:

```text
El módulo reexporta API.
Quieres documentar superficie pública.
Usas from module import * en casos controlados.
```

---

# Parte XII: herramientas

## 57. `pycodestyle`

`pycodestyle` verifica algunas convenciones de PEP 8.

Instalar:

```bash
pip install pycodestyle
```

Ejecutar:

```bash
pycodestyle src tests
```

Uso:

```text
Útil para revisar estilo PEP 8 clásico.
Limitado frente a herramientas modernas.
```

---

## 58. `flake8`

`flake8` combina chequeos de estilo, errores y plugins.

Instalar:

```bash
pip install flake8
```

Ejecutar:

```bash
flake8 src tests
```

Config ejemplo:

```ini
[flake8]
max-line-length = 88
extend-ignore = E203,W503
exclude =
    .git,
    .venv,
    __pycache__,
    build,
    dist
```

---

## 59. `black`

`black` formatea código automáticamente.

Instalar:

```bash
pip install black
```

Ejecutar:

```bash
black src tests
```

Verificar sin modificar:

```bash
black --check src tests
```

Config en `pyproject.toml`:

```toml
[tool.black]
line-length = 88
target-version = ["py312"]
```

Ventajas:

```text
Reduce discusiones de estilo.
Formato consistente.
Poca configuración.
```

---

## 60. `isort`

`isort` ordena imports.

Instalar:

```bash
pip install isort
```

Ejecutar:

```bash
isort src tests
```

Config compatible con black:

```toml
[tool.isort]
profile = "black"
line_length = 88
```

---

## 61. `ruff`

`ruff` puede actuar como linter, formatter, organizador de imports y reemplazo de varias herramientas tradicionales.

Instalar:

```bash
pip install ruff
```

Revisar:

```bash
ruff check src tests
```

Corregir automáticamente:

```bash
ruff check src tests --fix
```

Formatear:

```bash
ruff format src tests
```

Verificar formato:

```bash
ruff format --check src tests
```

Config recomendada:

```toml
[tool.ruff]
line-length = 88
target-version = "py312"

[tool.ruff.lint]
select = [
    "E",      # pycodestyle errors
    "F",      # pyflakes
    "W",      # pycodestyle warnings
    "I",      # isort
    "B",      # flake8-bugbear
    "UP",     # pyupgrade
    "SIM",    # flake8-simplify
]
ignore = [
    "E501",   # line length, handled by formatter
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
line-ending = "auto"
```

---

## 62. Configuración recomendada con Ruff

Para proyectos nuevos:

```toml
[project]
requires-python = ">=3.12"

[tool.ruff]
line-length = 88
target-version = "py312"
src = ["src", "tests"]

[tool.ruff.lint]
select = [
    "E",
    "F",
    "W",
    "I",
    "B",
    "UP",
    "SIM",
    "C4",
    "ARG",
    "PTH",
]
ignore = [
    "E501",
]

[tool.ruff.lint.per-file-ignores]
"tests/**/*.py" = [
    "ARG",
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
skip-magic-trailing-comma = false
line-ending = "auto"
```

Comandos:

```bash
ruff check src tests --fix
ruff format src tests
```

---

## 63. Pre-commit

`pre-commit` ejecuta validaciones antes de cada commit.

Instalar:

```bash
pip install pre-commit
```

Archivo:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.12.0
    hooks:
      - id: ruff-check
        args: [--fix]
      - id: ruff-format
```

Instalar hooks:

```bash
pre-commit install
```

Ejecutar en todo el repo:

```bash
pre-commit run --all-files
```

---

## 64. CI/CD

Ejemplo GitHub Actions:

```yaml
name: Python Style

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  style:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install tools
        run: |
          python -m pip install --upgrade pip
          pip install ruff

      - name: Lint
        run: ruff check src tests

      - name: Format check
        run: ruff format --check src tests
```

---

# Parte XIII: ejemplos antes/después

## 65. Ejemplo 1: formato general

Antes:

```python
import requests,os
from my_app.users import User
def getUser(id):
 if id==None:
  return None
 response=requests.get("https://api.example.com/users/"+str(id))
 if response.status_code==200:return User(**response.json())
 else: raise Exception("error")
```

Después:

```python
import os

import requests

from my_app.users import User


def get_user(user_id: int | None) -> User | None:
    if user_id is None:
        return None

    response = requests.get(
        f"https://api.example.com/users/{user_id}",
        timeout=10,
    )

    if response.status_code == 200:
        return User(**response.json())

    raise UserFetchError(f"Could not fetch user {user_id}")
```

Mejoras:

```text
Imports ordenados.
Nombre snake_case.
Comparación con None usando is.
Llamada larga separada.
Timeout explícito.
Excepción específica.
```

---

## 66. Ejemplo 2: función con muchos argumentos

Antes:

```python
def create_user(first,last,email,role,active,verified,send_email):
    ...
```

Después:

```python
@dataclass(frozen=True)
class CreateUserInput:
    first_name: str
    last_name: str
    email: str
    role: str
    is_active: bool = True
    is_verified: bool = False
    should_send_email: bool = True


def create_user(user_input: CreateUserInput) -> User:
    ...
```

---

## 67. Ejemplo 3: control de errores

Antes:

```python
try:
    data = json.loads(raw)
except:
    data = {}
```

Después:

```python
try:
    data = json.loads(raw)
except json.JSONDecodeError as exc:
    raise InvalidPayloadError("Payload is not valid JSON") from exc
```

---

## 68. Ejemplo 4: comprehensions complejas

Antes:

```python
result = [x.name.lower() for group in groups for x in group.items if x.active and not x.deleted and x.score > 10]
```

Después:

```python
result = []

for group in groups:
    for item in group.items:
        if item.active and not item.deleted and item.score > 10:
            result.append(item.name.lower())
```

O con helper:

```python
def is_eligible(item: Item) -> bool:
    return item.active and not item.deleted and item.score > 10


result = [
    item.name.lower()
    for group in groups
    for item in group.items
    if is_eligible(item)
]
```

---

# Parte XIV: excepciones razonables a PEP 8

## 69. Cuándo romper una regla

Puede ser razonable romper PEP 8 cuando:

```text
Mejora la legibilidad.
Mantiene consistencia con código existente.
Evita romper compatibilidad pública.
La herramienta del equipo usa otra convención documentada.
Hay restricciones externas.
```

Ejemplo:

```text
PEP 8 recomienda 79 caracteres.
Tu equipo usa black con 88.
Esto es aceptable si está automatizado y documentado.
```

---

## 70. Cuándo no romper reglas

No rompas reglas por:

```text
Pereza.
Preferencia personal no consensuada.
Desconocimiento.
Imitar estilos de otros lenguajes.
Evitar configurar herramientas.
```

---

# Parte XV: guía de estilo recomendada para equipos

## 71. Política mínima de equipo

```text
[ ] Usar Python >= 3.11 o versión definida.
[ ] Usar ruff check.
[ ] Usar ruff format o black.
[ ] Usar imports ordenados automáticamente.
[ ] Definir line-length.
[ ] Usar type hints en código nuevo.
[ ] Usar docstrings en API pública.
[ ] Bloquear CI si lint/format falla.
[ ] No discutir formato manualmente en code review.
```

---

## 72. Reglas de code review

En code review, evita comentarios manuales como:

```text
Falta espacio.
Línea muy larga.
Import mal ordenado.
Comillas distintas.
```

Eso debe automatizarse.

Enfoca review humano en:

```text
Correctitud.
Diseño.
Nombres.
Riesgos.
Tests.
Errores.
Seguridad.
Mantenibilidad.
```

---

## 73. Checklist PEP 8

```text
Formato
[ ] Indentación de 4 espacios.
[ ] Líneas dentro del límite acordado.
[ ] Espacios correctos alrededor de operadores.
[ ] Sin espacios innecesarios.
[ ] Líneas en blanco consistentes.

Imports
[ ] Imports al inicio.
[ ] Librería estándar, terceros y locales separados.
[ ] Sin wildcard imports.
[ ] Sin imports no usados.

Nombres
[ ] Funciones y variables en snake_case.
[ ] Clases en CapWords.
[ ] Constantes en UPPER_CASE.
[ ] Excepciones terminan en Error.
[ ] Booleanos con is_/has_/can_/should_ cuando aplica.

Funciones
[ ] Responsabilidad clara.
[ ] Retornos consistentes.
[ ] Argumentos legibles.
[ ] Keyword arguments cuando evitan ambigüedad.

Clases
[ ] Responsabilidad clara.
[ ] Métodos ordenados.
[ ] No usar clases innecesarias para funciones simples.

Comentarios y docstrings
[ ] Comentarios explican por qué.
[ ] API pública documentada.
[ ] Docstrings claros y breves.

Errores
[ ] Excepciones específicas.
[ ] No se silencian errores sin razón.
[ ] Se usa raise from cuando corresponde.

Herramientas
[ ] Ruff/Black/isort configurados.
[ ] Pre-commit opcional.
[ ] CI valida estilo.
```

---

# Parte XVI: configuración completa recomendada

## 74. `pyproject.toml` con Ruff

```toml
[project]
name = "my-python-project"
version = "0.1.0"
requires-python = ">=3.12"

[tool.ruff]
line-length = 88
target-version = "py312"
src = ["src", "tests"]

[tool.ruff.lint]
select = [
    "E",
    "F",
    "W",
    "I",
    "B",
    "UP",
    "SIM",
    "C4",
    "ARG",
    "PTH",
]
ignore = [
    "E501",
]

[tool.ruff.lint.per-file-ignores]
"tests/**/*.py" = [
    "ARG",
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
line-ending = "auto"
```

Comandos:

```bash
ruff check src tests --fix
ruff format src tests
```

---

## 75. `pyproject.toml` con Black + isort

```toml
[tool.black]
line-length = 88
target-version = ["py312"]

[tool.isort]
profile = "black"
line_length = 88
```

Comandos:

```bash
black src tests
isort src tests
```

---

## 76. `setup.cfg` con flake8

```ini
[flake8]
max-line-length = 88
extend-ignore = E203,W503
exclude =
    .git,
    .venv,
    __pycache__,
    build,
    dist
```

Comando:

```bash
flake8 src tests
```

---

# Parte XVII: comandos rápidos

## 77. Instalar herramientas

Ruff:

```bash
pip install ruff
```

Black + isort + flake8:

```bash
pip install black isort flake8
```

pycodestyle:

```bash
pip install pycodestyle
```

pre-commit:

```bash
pip install pre-commit
```

---

## 78. Ejecutar herramientas

Ruff:

```bash
ruff check src tests
ruff check src tests --fix
ruff format src tests
ruff format --check src tests
```

Black:

```bash
black src tests
black --check src tests
```

isort:

```bash
isort src tests
isort --check-only src tests
```

flake8:

```bash
flake8 src tests
```

pycodestyle:

```bash
pycodestyle src tests
```

pre-commit:

```bash
pre-commit install
pre-commit run --all-files
```

---

# Parte XVIII: resumen de reglas principales

```text
1. Usa 4 espacios por indentación.
2. No mezcles tabs y espacios.
3. Usa nombres snake_case para funciones y variables.
4. Usa CapWords para clases.
5. Usa UPPER_CASE para constantes.
6. Ordena imports: estándar, terceros, locales.
7. Evita wildcard imports.
8. Usa espacios alrededor de operadores.
9. Evita espacios dentro de paréntesis, corchetes o llaves.
10. Compara None con is / is not.
11. No compares booleanos con == True o == False.
12. Usa if items en vez de len(items) > 0.
13. Mantén funciones pequeñas y con responsabilidad clara.
14. Usa excepciones específicas.
15. No silencies errores sin razón.
16. Escribe comentarios que expliquen por qué.
17. Usa docstrings en API pública.
18. Usa type hints en código nuevo.
19. Automatiza formato con ruff format o black.
20. Automatiza linting con ruff, flake8 o pycodestyle.
21. Define line-length de equipo.
22. No discutas formato manualmente en code review.
23. Rompe PEP 8 solo con una razón clara.
```

---

# Fuentes de referencia recomendadas

```text
- PEP 8: Style Guide for Python Code.
- PEP 257: Docstring Conventions.
- pycodestyle documentation.
- Ruff documentation.
- Black documentation.
- isort documentation.
- flake8 documentation.
```
