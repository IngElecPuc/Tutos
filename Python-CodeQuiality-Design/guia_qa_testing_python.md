# Guía de buenas prácticas de QA y testing de sistemas con Python

## Objetivo

Esta guía resume buenas prácticas de QA, testing, debugging, logging, monitoreo y validación de sistemas con orientación a Python. Está pensada para aplicaciones backend, APIs, pipelines, servicios web, librerías internas y sistemas productivos.

Está organizada en tres niveles:

```text
1. Básico: conceptos, ambientes, unit tests, pytest, unittest, debugging, logs y smoke tests.
2. Intermedio: integración, APIs, fixtures, mocks, coverage, CI/CD, staging, monitoreo y estrategia de releases.
3. Avanzado: contract testing, property-based testing, E2E, performance, resiliencia, observabilidad, chaos testing, quality gates y auditoría de calidad.
```

La idea principal:

```text
QA no es solo escribir tests.
QA es diseñar procesos para reducir defectos, detectar riesgos temprano y operar sistemas con confianza.
```

---

# Parte I: fundamentos básicos

## 1. Qué es QA

QA significa Quality Assurance. No se limita a probar al final. Incluye prácticas para asegurar calidad durante todo el ciclo de vida del software.

QA incluye:

```text
Definir criterios de aceptación.
Revisar requerimientos.
Diseñar estrategia de pruebas.
Automatizar validaciones.
Revisar código.
Medir cobertura.
Revisar logs.
Monitorear producción.
Analizar incidentes.
Prevenir regresiones.
Documentar riesgos.
```

Testing es una parte de QA:

```text
QA       proceso amplio de calidad.
Testing  ejecución de pruebas específicas.
```

---

## 2. Objetivos del testing

Los tests deben responder preguntas concretas:

```text
¿Esta función calcula bien?
¿Este endpoint responde como esperamos?
¿Este flujo de usuario sigue funcionando?
¿El sistema maneja errores?
¿El sistema rechaza entradas inválidas?
¿El sistema se comporta bien bajo carga?
¿Una corrección rompió algo existente?
¿Podemos desplegar con confianza?
```

Un buen test no prueba “todo”. Prueba un comportamiento relevante.

---

## 3. Pirámide de testing

Modelo práctico:

```text
E2E tests
▲ pocos, lentos, flujos críticos completos

Integration tests
▲ cantidad media, prueban componentes juntos

Unit tests
▲ muchos, rápidos, prueban piezas pequeñas
```

Regla práctica:

```text
Ten muchos unit tests.
Ten suficientes integration tests.
Ten pocos E2E tests, pero cubriendo flujos críticos.
```

Antipatrón:

```text
Depender casi exclusivamente de E2E.
Son lentos, frágiles y difíciles de depurar.
```

---

## 4. Tipos de pruebas

Tipos frecuentes:

```text
Unit tests:
Prueban una función, clase o módulo aislado.

Integration tests:
Prueban interacción entre módulos, base de datos, cache, APIs internas.

Functional tests:
Prueban una funcionalidad desde la perspectiva del usuario o negocio.

End-to-end tests:
Prueban un flujo completo del sistema.

Smoke tests:
Pruebas rápidas para confirmar que el sistema arranca y lo básico funciona.

Regression tests:
Evitan que vuelva un bug ya corregido.

Contract tests:
Verifican que productor y consumidor respetan un contrato.

Performance tests:
Miden latencia, throughput y consumo de recursos.

Load tests:
Prueban comportamiento con carga esperada o alta.

Security tests:
Buscan fallas de seguridad.

Acceptance tests:
Validan criterios acordados con negocio.
```

---

## 5. Ambientes: development, staging/testing y production

Un sistema sano distingue ambientes.

### 5.1 Development

Ambiente local o de desarrollo.

Uso:

```text
Escribir código.
Ejecutar unit tests.
Probar cambios rápidos.
Depurar.
Usar datos falsos o anonimizados.
```

Características:

```text
Configuración flexible.
Logs detallados.
Debug habilitado.
Base de datos local o contenedorizada.
Dependencias simuladas cuando conviene.
```

Nunca debería contener:

```text
Credenciales productivas.
Datos reales sensibles sin anonimizar.
Acceso amplio a recursos críticos.
```

---

### 5.2 Staging / Testing

Ambiente similar a producción, pero sin afectar usuarios reales.

Uso:

```text
Validación previa al release.
Smoke tests.
Integration tests.
E2E tests.
Pruebas de migraciones.
Pruebas de configuración.
Pruebas con servicios reales o simulados controlados.
```

Características:

```text
Infraestructura parecida a producción.
Variables separadas.
Base de datos separada.
Credenciales separadas.
Logs y monitoreo activos.
Datos sintéticos o anonimizados.
```

Regla:

```text
Staging debe parecerse a producción lo suficiente para detectar problemas reales,
pero debe estar aislado para no dañar datos ni usuarios reales.
```

---

### 5.3 Production

Ambiente usado por usuarios reales.

Uso:

```text
Operación real.
Monitoreo.
Alertas.
Análisis de incidentes.
Validaciones no destructivas.
```

Características:

```text
Debug desactivado.
Logs estructurados.
Monitoreo y alertas.
Backups.
Control de acceso.
Secretos protegidos.
Cambios mediante CI/CD.
```

En producción no debes:

```text
Probar manualmente funciones destructivas sin control.
Ejecutar scripts ad hoc sin revisión.
Activar debug público.
Imprimir secretos en logs.
Usar datos reales para experimentos no autorizados.
```

---

## 6. Configuración por ambiente

Usa variables de entorno.

```env
APP_ENV=development
DATABASE_URL=postgresql://localhost/app_dev
LOG_LEVEL=DEBUG
```

Ejemplo en Python:

```python
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    app_env: str = "development"
    database_url: str
    log_level: str = "INFO"

    @property
    def is_production(self) -> bool:
        return self.app_env == "production"


settings = Settings()
```

Reglas:

```text
No hardcodees configuración.
No mezcles credenciales entre ambientes.
No uses APP_ENV para saltarte seguridad.
Valida configuración al arrancar.
```

---

## 7. Estructura básica de proyecto Python

Ejemplo:

```text
my_app/
    src/
        my_app/
            __init__.py
            config.py
            services.py
            models.py
            api.py
    tests/
        unit/
            test_services.py
        integration/
            test_api.py
        e2e/
            test_user_flow.py
    pyproject.toml
    README.md
```

Regla práctica:

```text
El código de producción vive en src/.
Los tests viven en tests/.
Los tests deben poder ejecutarse con un comando único.
```

---

## 8. Herramientas principales

Herramientas comunes en Python:

```text
pytest           framework de testing recomendado en muchos proyectos.
unittest         framework estándar incluido en Python.
coverage.py      medición de cobertura.
pytest-cov       integración de coverage con pytest.
pytest-mock      mocks integrados con pytest.
responses        mock de requests HTTP.
respx            mock de httpx.
freezegun        control de fechas y tiempo.
hypothesis       property-based testing.
playwright       E2E y pruebas de navegador.
locust           load testing.
ruff             linting y formato.
mypy / pyright   chequeo estático de tipos.
tox / nox        matrices de pruebas.
```

---

# Parte II: unit tests básicos

## 9. Primer unit test con pytest

Código:

```python
# src/my_app/calculator.py

def add(a: int, b: int) -> int:
    return a + b
```

Test:

```python
# tests/unit/test_calculator.py

from my_app.calculator import add


def test_add_two_numbers() -> None:
    result = add(2, 3)

    assert result == 5
```

Ejecutar:

```bash
pytest
```

Un buen unit test tiene:

```text
Nombre claro.
Entrada controlada.
Resultado esperado.
Sin dependencias externas innecesarias.
Ejecución rápida.
```

---

## 10. Patrón Arrange, Act, Assert

Estructura recomendada:

```python
def test_calculate_total_with_tax() -> None:
    # Arrange
    subtotal = 100
    tax_rate = 0.19

    # Act
    total = calculate_total(subtotal, tax_rate)

    # Assert
    assert total == 119
```

```text
Arrange: prepara datos.
Act: ejecuta comportamiento.
Assert: verifica resultado.
```

No siempre necesitas comentarios, pero la estructura mental ayuda.

---

## 11. Tests con `unittest`

`unittest` viene incluido en Python.

```python
import unittest

from my_app.calculator import add


class TestCalculator(unittest.TestCase):
    def test_add_two_numbers(self) -> None:
        self.assertEqual(add(2, 3), 5)


if __name__ == "__main__":
    unittest.main()
```

Ejecutar:

```bash
python -m unittest
```

Cuándo usar `unittest`:

```text
Proyectos estándar sin dependencias externas.
Compatibilidad con herramientas antiguas.
Equipos que ya lo usan.
```

Cuándo preferir `pytest`:

```text
Tests más concisos.
Fixtures poderosas.
Parametrización simple.
Mejor experiencia en proyectos grandes.
```

---

## 12. Nombres de tests

Mal nombre:

```python
def test_1() -> None:
    ...
```

Mejor:

```python
def test_create_user_rejects_invalid_email() -> None:
    ...
```

Convención:

```text
test_<comportamiento>_<condición>_<resultado>
```

Ejemplos:

```python
def test_login_with_wrong_password_returns_unauthorized() -> None:
    ...

def test_invoice_total_includes_tax() -> None:
    ...

def test_parser_ignores_empty_lines() -> None:
    ...
```

---

## 13. Probar errores

Con `pytest.raises`:

```python
import pytest

from my_app.users import validate_age


def test_validate_age_rejects_negative_age() -> None:
    with pytest.raises(ValueError, match="age must be positive"):
        validate_age(-1)
```

Regla:

```text
No pruebes solo el camino feliz.
Prueba errores esperados.
```

---

## 14. Parametrización

Evita duplicar tests.

```python
import pytest

from my_app.calculator import add


@pytest.mark.parametrize(
    ("a", "b", "expected"),
    [
        (1, 2, 3),
        (0, 0, 0),
        (-1, 1, 0),
        (10, -5, 5),
    ],
)
def test_add(a: int, b: int, expected: int) -> None:
    assert add(a, b) == expected
```

Ventajas:

```text
Más casos con menos código.
Más fácil agregar bordes.
Reporte claro por input.
```

---

## 15. Fixtures

Una fixture prepara datos o dependencias.

```python
import pytest


@pytest.fixture
def sample_user() -> dict[str, str]:
    return {
        "id": "user_123",
        "email": "user@example.com",
    }


def test_user_email(sample_user: dict[str, str]) -> None:
    assert sample_user["email"] == "user@example.com"
```

Fixture con setup/teardown:

```python
import pytest


@pytest.fixture
def temp_file(tmp_path):
    file_path = tmp_path / "data.txt"
    file_path.write_text("hello", encoding="utf-8")

    yield file_path

    # teardown opcional
```

Buenas prácticas:

```text
Fixtures pequeñas.
Nombres claros.
No esconder demasiada lógica.
Evitar fixtures globales mágicas.
```

---

## 16. `tmp_path`

Para archivos temporales:

```python
def test_write_report(tmp_path) -> None:
    report_path = tmp_path / "report.txt"

    write_report(report_path, "ok")

    assert report_path.read_text(encoding="utf-8") == "ok"
```

Ventaja:

```text
Cada test recibe un directorio temporal aislado.
Evita ensuciar el proyecto.
```

---

## 17. Mocks

Un mock reemplaza una dependencia.

Ejemplo con `unittest.mock`:

```python
from unittest.mock import Mock

from my_app.notifications import send_welcome_email


def test_send_welcome_email_calls_email_client() -> None:
    email_client = Mock()

    send_welcome_email(email_client, "user@example.com")

    email_client.send.assert_called_once_with(
        to="user@example.com",
        subject="Welcome",
        body="Thanks for joining.",
    )
```

Cuándo usar mocks:

```text
Servicios externos.
Email.
Pagos.
APIs lentas.
Reloj.
Sistema de archivos, si no quieres tocarlo.
```

Riesgos:

```text
Tests demasiado acoplados a implementación.
Mocks que no representan comportamiento real.
Falsa confianza si nunca haces integration tests.
```

---

## 18. `monkeypatch`

`monkeypatch` permite modificar variables, atributos o entorno temporalmente.

```python
def test_uses_environment_variable(monkeypatch) -> None:
    monkeypatch.setenv("APP_ENV", "testing")

    assert get_app_env() == "testing"
```

Mock de función:

```python
def test_generate_id(monkeypatch) -> None:
    monkeypatch.setattr(
        "my_app.ids.uuid4",
        lambda: "fixed-id",
    )

    assert generate_id() == "fixed-id"
```

Regla:

```text
Usa monkeypatch para aislar efectos globales.
No abuses para ocultar mal diseño.
```

---

# Parte III: debugging, logs y smoke tests

## 19. Debugging básico

Debugging no es adivinar. Es observar el estado del programa.

Técnicas:

```text
Leer stack traces.
Reproducir el bug.
Reducir el caso.
Usar logs.
Usar breakpoints.
Usar pdb.
Agregar tests que reproduzcan el bug.
```

Flujo recomendado:

```text
1. Reproducir.
2. Aislar.
3. Entender causa.
4. Corregir.
5. Agregar test de regresión.
6. Verificar.
```

---

## 20. `pdb`

Python incluye `pdb`.

```python
def calculate_discount(price: float, percentage: float) -> float:
    breakpoint()
    return price * (1 - percentage)
```

Ejecutar el código y usar comandos:

```text
n      next
s      step
c      continue
p var  print variable
l      list code
q      quit
```

Regla:

```text
No commitees breakpoints accidentales.
```

Puedes detectarlos con linting o revisión de código.

---

## 21. Logging básico

Usa `logging`, no `print`, para sistemas reales.

```python
import logging

logger = logging.getLogger(__name__)


def create_user(email: str) -> None:
    logger.info("Creating user", extra={"email": email})
```

Configuración básica:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s %(message)s",
)
```

Niveles:

```text
DEBUG     detalle para desarrollo.
INFO      eventos normales relevantes.
WARNING   situación inesperada, pero no fatal.
ERROR     operación fallida.
CRITICAL  falla grave del sistema.
```

---

## 22. Logs estructurados

En producción conviene usar JSON o estructura consistente.

Ejemplo conceptual:

```json
{
    "timestamp": "2026-07-04T12:00:00Z",
    "level": "INFO",
    "event": "user_created",
    "user_id": "user_123",
    "request_id": "req_abc",
    "service": "api"
}
```

Buenas prácticas:

```text
Incluye request_id o correlation_id.
Incluye usuario/tenant cuando sea seguro.
No incluyas contraseñas, tokens ni secretos.
Usa niveles correctamente.
Registra eventos de negocio importantes.
Registra errores con stack trace.
```

Ejemplo:

```python
try:
    process_payment(payment_id)
except PaymentError:
    logger.exception(
        "Payment processing failed",
        extra={"payment_id": payment_id},
    )
    raise
```

`logger.exception` debe usarse dentro de un `except`; incluye stack trace.

---

## 23. Qué no loguear

No loguees:

```text
Contraseñas.
Tokens.
Cookies de sesión.
API keys.
Refresh tokens.
Números completos de tarjetas.
Datos personales sensibles.
Documentos privados.
Secretos de infraestructura.
```

Mala práctica:

```python
logger.info("Login payload: %s", payload)
```

Mejor:

```python
logger.info(
    "Login attempt",
    extra={
        "email_hash": hash_email(email),
        "ip": client_ip,
    },
)
```

---

## 24. Smoke tests

Smoke tests validan que el sistema básico funciona después de deploy.

Deben ser:

```text
Rápidos.
Pocos.
No destructivos o fácilmente reversibles.
Orientados a salud del sistema.
Ejecutables en staging y producción.
```

Ejemplos:

```text
GET /health devuelve 200.
GET /version devuelve versión esperada.
Login de cuenta sintética funciona.
Endpoint crítico responde.
Base de datos accesible.
Cola accesible.
Servicio externo mockeado o disponible.
```

Ejemplo con `requests`:

```python
import requests


def test_healthcheck() -> None:
    response = requests.get("https://api.example.com/health", timeout=5)

    assert response.status_code == 200
    assert response.json()["status"] == "ok"
```

---

## 25. Health checks

Un endpoint de health check básico:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/health")
def health() -> dict[str, str]:
    return {"status": "ok"}
```

Health check más completo:

```python
@app.get("/ready")
def ready() -> dict[str, str]:
    check_database()
    check_cache()

    return {"status": "ready"}
```

Diferencia práctica:

```text
/health:
El proceso está vivo.

/ready:
El servicio puede recibir tráfico real.
```

No hagas un health check tan pesado que se convierta en problema de rendimiento.

---

# Parte IV: nivel intermedio

## 26. Integration tests

Un integration test prueba varios componentes juntos.

Ejemplo:

```text
API + base de datos.
Servicio + cola.
Repositorio + PostgreSQL.
Cliente HTTP + API externa simulada.
```

Ejemplo con FastAPI:

```python
from fastapi.testclient import TestClient

from my_app.main import app

client = TestClient(app)


def test_create_task() -> None:
    response = client.post(
        "/tasks",
        json={
            "title": "Estudiar pytest",
        },
    )

    assert response.status_code == 201
    assert response.json()["title"] == "Estudiar pytest"
```

Buenas prácticas:

```text
Usar base de datos de testing.
Limpiar datos entre tests.
No depender del orden de tests.
No llamar servicios externos reales salvo pruebas específicas.
```

---

## 27. Base de datos para tests

Opciones:

```text
SQLite en memoria:
Rápido, pero puede diferir de PostgreSQL.

PostgreSQL en Docker:
Más realista.

Testcontainers:
Levanta dependencias reales por test suite.

Schema temporal:
Útil en integración avanzada.

Transacciones por test:
Rápidas y reversibles.
```

Recomendación:

```text
Para lógica pura, unit tests.
Para SQL específico de PostgreSQL, usa PostgreSQL real en tests.
```

Ejemplo conceptual con fixture:

```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import Session


@pytest.fixture
def db_session() -> Session:
    engine = create_engine("postgresql+psycopg://test:test@localhost/test_db")

    with Session(engine) as session:
        yield session
        session.rollback()
```

---

## 28. Tests de APIs

Verifica:

```text
Status code.
Body.
Headers.
Errores.
Autenticación.
Autorización.
Validaciones.
Paginación.
Idempotencia.
```

Ejemplo:

```python
def test_get_missing_task_returns_404(client) -> None:
    response = client.get("/tasks/not-found")

    assert response.status_code == 404
    assert response.json() == {"detail": "Task not found"}
```

Test de validación:

```python
def test_create_task_rejects_empty_title(client) -> None:
    response = client.post("/tasks", json={"title": ""})

    assert response.status_code == 422
```

---

## 29. Tests de autorización

Los errores de autorización son críticos.

```python
def test_user_cannot_read_other_users_task(
    client,
    user_a_token: str,
    user_b_task_id: str,
) -> None:
    response = client.get(
        f"/tasks/{user_b_task_id}",
        headers={"Authorization": f"Bearer {user_a_token}"},
    )

    assert response.status_code in {403, 404}
```

Regla:

```text
Todo endpoint con datos privados debe tener tests de acceso cruzado.
```

Casos mínimos:

```text
Usuario anónimo.
Usuario autenticado sin permisos.
Usuario dueño.
Usuario admin.
Usuario de otro tenant.
```

---

## 30. Mocks de APIs externas

No llames APIs reales en unit tests.

Ejemplo con `responses` para `requests`:

```python
import responses

from my_app.github import get_repo


@responses.activate
def test_get_repo() -> None:
    responses.add(
        responses.GET,
        "https://api.github.com/repos/org/repo",
        json={"name": "repo"},
        status=200,
    )

    repo = get_repo("org", "repo")

    assert repo["name"] == "repo"
```

Para `httpx`, puedes usar `respx`.

Regla:

```text
Mockea límites externos.
Prueba integraciones reales en suites separadas y controladas.
```

---

## 31. Test doubles

Tipos:

```text
Dummy:
Objeto necesario pero no usado.

Stub:
Devuelve respuestas predefinidas.

Mock:
Verifica llamadas.

Fake:
Implementación simple pero funcional.

Spy:
Registra cómo fue usado.
```

Ejemplo de fake:

```python
class FakeEmailSender:
    def __init__(self) -> None:
        self.sent_emails = []

    def send(self, to: str, subject: str, body: str) -> None:
        self.sent_emails.append({
            "to": to,
            "subject": subject,
            "body": body,
        })


def test_register_user_sends_email() -> None:
    email_sender = FakeEmailSender()

    register_user("user@example.com", email_sender)

    assert len(email_sender.sent_emails) == 1
```

Fakes suelen ser más mantenibles que mocks excesivos.

---

## 32. Coverage

Coverage mide qué líneas o ramas ejecutan los tests.

Instalar:

```bash
pip install pytest-cov
```

Ejecutar:

```bash
pytest --cov=src --cov-report=term-missing
```

Con umbral:

```bash
pytest --cov=src --cov-fail-under=85
```

En `pyproject.toml`:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]

[tool.coverage.run]
branch = true
source = ["src"]

[tool.coverage.report]
show_missing = true
fail_under = 85
```

Advertencia:

```text
Alta cobertura no garantiza calidad.
Baja cobertura suele indicar riesgo.
Mide cobertura de líneas y de ramas.
```

---

## 33. Qué testear primero

Prioriza:

```text
Lógica de negocio crítica.
Cálculos financieros.
Autorización.
Validaciones.
Transformaciones de datos.
Integraciones frágiles.
Bugs corregidos.
Migraciones.
Flujos de usuario críticos.
```

Evita gastar demasiado al inicio en:

```text
Getters triviales.
Código generado.
Wrappers sin lógica.
Detalles de implementación.
```

---

## 34. Tests de regresión

Cada bug importante debería dejar un test.

Flujo:

```text
1. Reproduce bug con test fallido.
2. Corrige.
3. Verifica que el test pasa.
4. Deja el test en la suite.
```

Ejemplo:

```python
def test_discount_never_returns_negative_total() -> None:
    total = apply_discount(price=100, discount=200)

    assert total == 0
```

---

## 35. Markers de pytest

Define categorías.

```toml
[tool.pytest.ini_options]
markers = [
    "unit: fast isolated tests",
    "integration: tests that require services",
    "e2e: end-to-end tests",
    "slow: slow tests",
    "smoke: production or staging smoke tests",
]
```

Uso:

```python
import pytest


@pytest.mark.integration
def test_database_connection() -> None:
    ...
```

Ejecutar solo unit:

```bash
pytest -m unit
```

Excluir lentos:

```bash
pytest -m "not slow"
```

---

## 36. CI/CD básico

Pipeline mínimo:

```text
1. Instalar dependencias.
2. Ejecutar lint.
3. Ejecutar type check.
4. Ejecutar unit tests.
5. Ejecutar integration tests.
6. Generar coverage.
7. Construir artefacto.
8. Desplegar a staging.
9. Ejecutar smoke tests.
10. Aprobar producción.
```

Ejemplo GitHub Actions:

```yaml
name: Python QA

on:
    pull_request:
    push:
        branches:
            - main

jobs:
    test:
        runs-on: ubuntu-latest

        steps:
            - uses: actions/checkout@v4

            - uses: actions/setup-python@v5
              with:
                  python-version: "3.12"

            - name: Install dependencies
              run: |
                  python -m pip install --upgrade pip
                  pip install -e ".[dev]"

            - name: Lint
              run: ruff check .

            - name: Format check
              run: ruff format --check .

            - name: Type check
              run: mypy src

            - name: Tests
              run: pytest --cov=src --cov-report=xml
```

---

## 37. Quality gates

Un quality gate bloquea cambios que no cumplen mínimos.

Ejemplos:

```text
Tests pasan.
Coverage no baja bajo umbral.
No hay errores de lint.
No hay errores de tipo.
No hay secretos detectados.
No hay vulnerabilidades críticas conocidas.
Smoke tests pasan en staging.
Revisión de código aprobada.
```

Regla:

```text
Un quality gate debe ser exigente, pero no tan frágil que el equipo lo ignore.
```

---

## 38. Staging y estrategia de release

Antes de producción:

```text
Desplegar a staging.
Ejecutar migraciones.
Correr smoke tests.
Correr E2E críticos.
Revisar logs.
Revisar métricas básicas.
Validar feature flags.
Aprobar release.
```

Checklist:

```text
[ ] Versión desplegada coincide con commit esperado.
[ ] Migraciones aplicadas.
[ ] Health checks OK.
[ ] Smoke tests OK.
[ ] Logs sin errores nuevos.
[ ] Métricas normales.
[ ] Rollback definido.
```

---

## 39. Feature flags

Permiten activar/desactivar funcionalidades sin redeploy.

Usos:

```text
Rollout gradual.
Pruebas A/B.
Apagar features defectuosas.
Activar por tenant.
Activar solo en staging.
```

Ejemplo:

```python
def is_new_checkout_enabled(user_id: str) -> bool:
    return feature_flags.is_enabled("new_checkout", user_id=user_id)
```

Tests:

```python
def test_old_checkout_when_flag_disabled(monkeypatch) -> None:
    monkeypatch.setattr(
        "my_app.feature_flags.is_enabled",
        lambda *args, **kwargs: False,
    )

    assert get_checkout_version("user_1") == "old"
```

Reglas:

```text
Documenta flags.
Define fecha de eliminación.
No acumules flags muertas.
```

---

# Parte V: monitoreo y observabilidad

## 40. Monitoreo

Monitoreo responde:

```text
¿El sistema está funcionando?
¿Está rápido?
¿Está fallando?
¿A quién afecta?
¿Desde cuándo?
```

Mínimos:

```text
Disponibilidad.
Latencia.
Tasa de errores.
Uso de CPU.
Uso de memoria.
Uso de disco.
Conexiones a DB.
Colas pendientes.
Jobs fallidos.
```

---

## 41. Las cuatro señales doradas

Para servicios web:

```text
Latency:
Cuánto tarda.

Traffic:
Cuánta demanda recibe.

Errors:
Qué porcentaje falla.

Saturation:
Qué tan cerca está de sus límites.
```

Ejemplos:

```text
p95 latency > 500 ms.
error rate > 1%.
CPU > 85%.
queue lag > 5 minutos.
```

---

## 42. Métricas

Ejemplo conceptual con Prometheus:

```python
from prometheus_client import Counter, Histogram

REQUEST_COUNT = Counter(
    "http_requests_total",
    "Total HTTP requests",
    ["method", "endpoint", "status"],
)

REQUEST_LATENCY = Histogram(
    "http_request_duration_seconds",
    "HTTP request latency",
    ["method", "endpoint"],
)
```

Buenas prácticas:

```text
No uses labels de cardinalidad infinita.
No pongas user_id como label.
Agrega endpoint normalizado, no URL completa con IDs.
Define SLOs.
```

---

## 43. Trazas

Las trazas ayudan a seguir una request entre servicios.

Una traza puede mostrar:

```text
Frontend -> API -> DB -> Servicio externo -> Cola
```

OpenTelemetry permite instrumentar Python para métricas, logs y trazas.

Conceptos:

```text
Trace:
Recorrido completo.

Span:
Una operación dentro de la traza.

Trace ID:
Identificador global de la traza.

Span ID:
Identificador de una operación.
```

Uso típico:

```text
Diagnosticar latencia.
Encontrar servicios lentos.
Correlacionar errores.
Seguir requests distribuidas.
```

---

## 44. Correlation ID

Agrega un ID por request.

```python
import uuid


def create_request_id() -> str:
    return str(uuid.uuid4())
```

Inclúyelo en:

```text
Logs.
Errores.
Respuestas.
Trazas.
Métricas si aplica con cuidado.
```

Ejemplo de header:

```http
X-Request-ID: req_abc123
```

Regla:

```text
Sin correlation_id, depurar producción distribuida es mucho más difícil.
```

---

## 45. Alertas

Una buena alerta debe ser accionable.

Mala alerta:

```text
CPU alta.
```

Mejor:

```text
API production: error rate 5xx > 2% durante 5 minutos.
Impacto probable: usuarios no pueden completar checkout.
Acción: revisar dashboard X y logs por request_id.
```

Evita alert fatigue:

```text
No alertar por ruido.
Agrupar síntomas.
Priorizar impacto usuario.
Definir severidad.
Documentar runbooks.
```

---

## 46. Runbooks

Un runbook documenta qué hacer ante una alerta.

Ejemplo:

```markdown
# Alerta: API error rate alto

## Síntoma

5xx > 2% durante 5 minutos.

## Primeras acciones

1. Revisar dashboard de latencia y errores.
2. Revisar despliegues recientes.
3. Revisar logs por endpoint.
4. Revisar base de datos.
5. Si empezó tras deploy, considerar rollback.

## Comandos útiles

- kubectl get pods
- kubectl logs ...
- pytest -m smoke --base-url=https://staging.example.com

## Escalamiento

Contactar backend on-call.
```

---

# Parte VI: nivel avanzado

## 47. Contract testing

Los contract tests verifican acuerdos entre productor y consumidor.

Ejemplo:

```text
Frontend espera:
GET /tasks retorna lista con id, title, completed.

Backend promete:
id: string
title: string
completed: boolean
```

Contrato con Pydantic:

```python
from pydantic import BaseModel


class TaskResponse(BaseModel):
    id: str
    title: str
    completed: bool
```

Test:

```python
def test_task_response_contract(client) -> None:
    response = client.get("/tasks")

    assert response.status_code == 200

    for item in response.json():
        TaskResponse.model_validate(item)
```

Útil para:

```text
Microservicios.
Frontend/backend separados.
APIs públicas.
Integraciones con proveedores.
```

---

## 48. Property-based testing

Property-based testing prueba propiedades generales con muchos inputs generados.

Instalar:

```bash
pip install hypothesis
```

Ejemplo:

```python
from hypothesis import given
from hypothesis import strategies as st


def reverse_twice(value: str) -> str:
    return value[::-1][::-1]


@given(st.text())
def test_reverse_twice_returns_original(value: str) -> None:
    assert reverse_twice(value) == value
```

Ejemplo para ordenar:

```python
from hypothesis import given
from hypothesis import strategies as st


@given(st.lists(st.integers()))
def test_sorted_result_is_ordered(values: list[int]) -> None:
    result = sorted(values)

    assert result == sorted(result)
```

Útil para:

```text
Parsers.
Serialización/deserialización.
Funciones matemáticas.
Normalización.
Validaciones.
Transformaciones de datos.
```

Regla:

```text
No escribas solo ejemplos.
Escribe invariantes.
```

---

## 49. Mutation testing

Mutation testing introduce pequeños cambios en el código para ver si los tests fallan.

Idea:

```text
Si cambio > por >= y los tests siguen pasando,
quizá los tests no detectan errores importantes.
```

Herramientas:

```text
mutmut.
cosmic-ray.
```

Uso:

```text
Código crítico.
Librerías.
Cálculos financieros.
Reglas de negocio complejas.
```

No suele ejecutarse en cada commit porque puede ser lento.

---

## 50. E2E tests con Playwright

Instalar:

```bash
pip install pytest-playwright
playwright install
```

Ejemplo:

```python
def test_homepage_has_title(page) -> None:
    page.goto("https://staging.example.com")

    assert page.get_by_role("heading", name="Inicio").is_visible()
```

Login:

```python
def test_user_can_login(page) -> None:
    page.goto("https://staging.example.com/login")

    page.get_by_label("Email").fill("test@example.com")
    page.get_by_label("Password").fill("correct-password")
    page.get_by_role("button", name="Entrar").click()

    assert page.get_by_role("heading", name="Dashboard").is_visible()
```

Buenas prácticas:

```text
Usa selectores por rol o label.
Evita selectores CSS frágiles.
No dependas de sleeps fijos.
Usa usuarios sintéticos.
Limpia datos.
Cubre flujos críticos, no todos los detalles.
```

---

## 51. Performance testing

Preguntas:

```text
¿Cuántas requests por segundo soporta?
¿Cuál es la latencia p95?
¿Qué pasa con 100 usuarios concurrentes?
¿Qué endpoint se degrada primero?
¿Qué recurso se satura?
```

Herramientas:

```text
Locust.
k6.
JMeter.
wrk.
hey.
```

Ejemplo conceptual con Locust:

```python
from locust import HttpUser, between, task


class WebsiteUser(HttpUser):
    wait_time = between(1, 3)

    @task
    def list_tasks(self) -> None:
        self.client.get("/tasks")
```

Métricas a observar:

```text
p50, p95, p99.
Throughput.
Error rate.
CPU.
Memoria.
DB connections.
Query latency.
Queue lag.
```

---

## 52. Stress, soak y spike tests

```text
Load test:
Carga esperada.

Stress test:
Carga sobre el límite para ver dónde rompe.

Soak test:
Carga moderada por mucho tiempo para detectar fugas.

Spike test:
Subidas repentinas de tráfico.
```

Uso:

```text
Antes de campañas.
Antes de lanzamientos.
Después de cambios de arquitectura.
Para sistemas con tráfico variable.
```

---

## 53. Resilience testing

Prueba cómo se comporta el sistema ante fallos.

Casos:

```text
Base de datos lenta.
API externa caída.
Timeouts.
Cola saturada.
Disco lleno.
Servicio reiniciado.
Red intermitente.
```

Buenas prácticas:

```text
Timeouts.
Retries con backoff.
Circuit breakers.
Fallbacks.
Idempotencia.
Dead-letter queues.
```

Test conceptual:

```python
def test_payment_provider_timeout_returns_retryable_error(monkeypatch) -> None:
    def timeout_request(*args, **kwargs):
        raise TimeoutError("provider timeout")

    monkeypatch.setattr("my_app.payments.call_provider", timeout_request)

    result = process_payment("payment_123")

    assert result.status == "retryable_failure"
```

---

## 54. Chaos testing

Chaos testing introduce fallos controlados para validar resiliencia.

Ejemplos:

```text
Matar pods.
Aumentar latencia.
Cortar red.
Bloquear dependencia.
Simular errores 500.
```

Reglas:

```text
Hazlo primero en staging.
Define hipótesis.
Define blast radius.
Ten rollback.
Monitorea.
Documenta resultados.
```

Ejemplo de hipótesis:

```text
Si una réplica de la API cae,
el balanceador debe enrutar tráfico a réplicas sanas
y el error rate no debe superar 1%.
```

---

## 55. Testing de migraciones

Las migraciones pueden romper producción.

Pruebas:

```text
Aplicar migración en DB vacía.
Aplicar migración en copia representativa.
Verificar rollback si existe.
Medir tiempo.
Verificar locks.
Verificar compatibilidad con versión anterior de app.
```

Checklist:

```text
[ ] Migración probada en staging.
[ ] Backup antes de producción.
[ ] Tiempo estimado conocido.
[ ] Locks evaluados.
[ ] App antigua y nueva compatibles durante deploy.
[ ] Rollback documentado.
```

---

## 56. Testing de datos y pipelines

Para ETL/ELT o pipelines:

Prueba:

```text
Esquema de entrada.
Esquema de salida.
Nulls.
Duplicados.
Rangos.
Referencialidad.
Idempotencia.
Volumen.
Datos tardíos.
Reprocesos.
```

Ejemplo:

```python
def test_transform_orders_removes_duplicates() -> None:
    raw_orders = [
        {"id": "1", "amount": 100},
        {"id": "1", "amount": 100},
    ]

    result = transform_orders(raw_orders)

    assert len(result) == 1
```

Validación con Pandas:

```python
def test_total_amount_is_non_negative(transformed_orders) -> None:
    assert (transformed_orders["amount"] >= 0).all()
```

Regla:

```text
Los pipelines deben ser idempotentes.
Reprocesar el mismo input no debería duplicar datos.
```

---

## 57. Snapshot testing

Snapshot testing compara salida actual contra una versión aprobada.

Útil para:

```text
JSON de respuesta.
HTML generado.
Configuraciones.
Serializaciones.
Documentos.
```

Riesgo:

```text
Aprobar snapshots sin revisarlos.
Snapshots enormes.
Tests frágiles ante cambios irrelevantes.
```

Regla:

```text
Los snapshots deben ser pequeños y revisables.
```

---

## 58. Flaky tests

Un flaky test falla de forma no determinista.

Causas:

```text
Dependencia del tiempo.
Orden de tests.
Estado compartido.
Red.
Servicios externos.
Sleeps fijos.
Concurrencia.
Datos globales.
```

Mitigaciones:

```text
Aislar estado.
Usar fixtures limpias.
Congelar tiempo.
Mockear red.
Esperas explícitas.
Eliminar dependencia de orden.
Reintentos solo como contención temporal.
```

Regla:

```text
Un flaky test es deuda técnica.
No lo normalices.
```

---

## 59. Test data management

Problemas comunes:

```text
Tests dependen de datos manuales.
Datos compartidos entre tests.
Datos productivos sensibles en staging.
IDs hardcodeados.
Orden implícito.
```

Buenas prácticas:

```text
Factories.
Fixtures.
Datos sintéticos.
Datos anonimizados.
Limpieza automática.
Transacciones por test.
IDs generados.
```

Ejemplo con factory simple:

```python
def make_task(
    title: str = "Default task",
    completed: bool = False,
) -> dict[str, object]:
    return {
        "id": "task_123",
        "title": title,
        "completed": completed,
    }
```

---

## 60. Testability: diseñar para probar

Código difícil de probar suele tener:

```text
Dependencias globales.
IO mezclado con lógica.
Tiempo real hardcodeado.
Configuración dispersa.
Funciones enormes.
Efectos secundarios ocultos.
```

Mejor diseño:

```text
Separar lógica pura de IO.
Inyectar dependencias.
Centralizar configuración.
Usar interfaces simples.
Retornar resultados explícitos.
```

Menos testeable:

```python
def send_report() -> None:
    data = requests.get("https://api.example.com/data").json()
    content = render_report(data)
    smtplib.SMTP("smtp.example.com").sendmail(...)
```

Más testeable:

```python
def build_report(data: list[dict]) -> str:
    return render_report(data)


def send_report(
    data_client,
    email_sender,
) -> None:
    data = data_client.get_data()
    content = build_report(data)
    email_sender.send(content)
```

---

# Parte VII: estrategia QA integral

## 61. Definir criterios de aceptación

Antes de desarrollar:

```text
Dado un contexto,
cuando ocurre una acción,
entonces debe pasar un resultado.
```

Ejemplo:

```text
Dado un usuario autenticado con tareas pendientes,
cuando marca una tarea como completada,
entonces la tarea aparece como completada en la lista
y el contador de pendientes disminuye en uno.
```

Esto se puede convertir en:

```text
Unit test.
Integration test.
E2E test.
Caso manual.
```

---

## 62. Definition of Done

Una tarea no está lista solo porque “funciona en local”.

Ejemplo:

```text
[ ] Código implementado.
[ ] Unit tests agregados.
[ ] Integration tests si aplica.
[ ] Validaciones cubiertas.
[ ] Logs relevantes agregados.
[ ] Errores manejados.
[ ] Documentación actualizada.
[ ] Feature flag si aplica.
[ ] Revisión de código aprobada.
[ ] CI pasa.
[ ] Smoke tests en staging pasan.
```

---

## 63. Estrategia de pruebas por riesgo

No todo requiere el mismo esfuerzo.

Riesgo alto:

```text
Pagos.
Autorización.
Datos personales.
Migraciones.
Borrado de datos.
Reportes regulatorios.
```

Requiere:

```text
Unit tests.
Integration tests.
E2E crítico.
Revisión manual.
Monitoreo.
Rollback.
```

Riesgo bajo:

```text
Cambio visual menor.
Texto.
Ordenamiento no crítico.
```

Puede requerir:

```text
Review.
Test manual rápido.
Snapshot o visual test si aplica.
```

---

## 64. Matriz de pruebas

Ejemplo:

| Área | Unit | Integration | E2E | Smoke | Monitoring |
|---|---:|---:|---:|---:|---:|
| Login | Sí | Sí | Sí | Sí | Sí |
| Crear tarea | Sí | Sí | Sí | Sí | Sí |
| Cambiar tema | Sí | No | No | No | No |
| Exportar reporte | Sí | Sí | Opcional | Sí | Sí |
| Migración DB | No | Sí | No | No | Sí |
| Autorización | Sí | Sí | Sí | Sí | Sí |

---

## 65. Reporte de QA

Un reporte útil incluye:

```text
Versión evaluada.
Ambiente.
Fecha.
Alcance.
Tests ejecutados.
Resultados.
Defectos encontrados.
Riesgos abiertos.
Recomendación: aprobar / bloquear / aprobar con riesgo.
```

Ejemplo:

```markdown
# Reporte QA - Release 1.8.0

Ambiente: staging
Commit: abc123
Fecha: 2026-07-04

## Resultado

Aprobado con observaciones.

## Ejecutado

- Unit tests: OK
- Integration tests: OK
- E2E críticos: OK
- Smoke tests: OK
- Migración: OK

## Riesgos

- El endpoint de reportes tuvo p95 de 900 ms, dentro del límite temporal pero debe monitorearse.

## Decisión

Puede pasar a producción con monitoreo reforzado.
```

---

# Parte VIII: checklists

## 66. Checklist de unit tests

```text
[ ] Test tiene nombre claro.
[ ] Prueba un comportamiento específico.
[ ] No depende de red.
[ ] No depende de orden.
[ ] No usa datos productivos.
[ ] Incluye caminos de error.
[ ] Incluye bordes relevantes.
[ ] Es rápido.
```

---

## 67. Checklist de integration tests

```text
[ ] Usa dependencias controladas.
[ ] Limpia datos.
[ ] Verifica status codes y payloads.
[ ] Cubre auth/autorización.
[ ] Cubre errores esperados.
[ ] No depende de servicios externos no controlados.
[ ] Puede ejecutarse en CI.
```

---

## 68. Checklist de smoke tests

```text
[ ] Health check OK.
[ ] Readiness OK.
[ ] Login sintético OK.
[ ] Endpoint crítico OK.
[ ] DB accesible.
[ ] Cache/cola accesible si aplica.
[ ] Versión desplegada correcta.
[ ] No produce datos irreversibles.
```

---

## 69. Checklist de producción

```text
[ ] Debug apagado.
[ ] Logs estructurados.
[ ] No se loguean secretos.
[ ] Métricas activas.
[ ] Trazas activas si aplica.
[ ] Alertas configuradas.
[ ] Backups activos.
[ ] Rollback definido.
[ ] Feature flags revisadas.
[ ] Smoke tests post-deploy.
```

---

## 70. Checklist de CI/CD

```text
[ ] Lint.
[ ] Format check.
[ ] Type check.
[ ] Unit tests.
[ ] Integration tests.
[ ] Coverage.
[ ] Dependency scan.
[ ] Secret scan.
[ ] Build.
[ ] Deploy staging.
[ ] Smoke staging.
[ ] Aprobación producción.
[ ] Smoke producción.
```

---

# Parte IX: configuración recomendada

## 71. `pyproject.toml` base

```toml
[project]
name = "my-app"
version = "0.1.0"
requires-python = ">=3.12"

[project.optional-dependencies]
dev = [
    "pytest",
    "pytest-cov",
    "ruff",
    "mypy",
    "hypothesis",
    "pytest-playwright",
]

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = [
    "--strict-markers",
    "--strict-config",
]
markers = [
    "unit: fast isolated tests",
    "integration: tests that require external dependencies",
    "e2e: end-to-end tests",
    "smoke: smoke tests",
    "slow: slow tests",
]

[tool.coverage.run]
branch = true
source = ["src"]

[tool.coverage.report]
show_missing = true
fail_under = 85

[tool.ruff]
line-length = 88

[tool.mypy]
python_version = "3.12"
strict = true
```

---

## 72. Comandos útiles

```bash
# Todos los tests
pytest

# Solo unit tests
pytest -m unit

# Excluir lentos
pytest -m "not slow"

# Coverage
pytest --cov=src --cov-report=term-missing

# Un archivo
pytest tests/unit/test_calculator.py

# Un test específico
pytest tests/unit/test_calculator.py::test_add_two_numbers

# Mostrar prints/logs
pytest -s

# Detener al primer fallo
pytest -x

# Ver los tests más lentos
pytest --durations=10
```

---

# Parte X: resumen de reglas principales

```text
1. QA es proceso; testing es una herramienta dentro de QA.
2. Separa development, staging/testing y production.
3. Usa datos y secretos distintos por ambiente.
4. Escribe muchos unit tests rápidos.
5. Usa integration tests para dependencias reales importantes.
6. Usa E2E solo para flujos críticos.
7. Agrega smoke tests para deploys.
8. No dependas de prints: usa logging.
9. No loguees secretos ni datos sensibles.
10. Usa coverage como señal, no como garantía absoluta.
11. Todo bug importante debe dejar un regression test.
12. Todo endpoint privado debe tener tests de autorización.
13. Usa CI/CD con quality gates.
14. Staging debe parecerse a producción.
15. Monitorea producción con métricas, logs y alertas.
16. Diseña el código para ser testeable.
17. Los flaky tests son deuda técnica.
18. Las migraciones también se prueban.
19. La calidad se mide por riesgo reducido, no por cantidad de tests.
20. Un release sin rollback ni monitoreo no está completamente listo.
```

---

# Fuentes de referencia recomendadas

```text
- Python Docs: unittest.
- Python Docs: logging.
- Python Docs: pdb.
- pytest documentation.
- pytest fixtures and parametrization.
- coverage.py documentation.
- pytest-cov documentation.
- Playwright Python documentation.
- Hypothesis documentation.
- OpenTelemetry Python documentation.
- GitHub Actions: Building and testing Python.
```
