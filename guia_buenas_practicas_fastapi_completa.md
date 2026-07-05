# Guía de buenas prácticas para APIs con FastAPI

## Objetivo

Esta guía resume buenas prácticas para diseñar, implementar y mantener APIs con FastAPI. Está pensada para proyectos REST sencillos o medianos, como APIs de juegos, modelos LLM, usuarios, libros, productos u otros recursos.

La guía está organizada de forma progresiva:

```text
1. Fundamentos de diseño de rutas y métodos HTTP.
2. Validación de datos y parámetros.
3. Modelos de entrada, salida y actualización.
4. Respuestas, códigos de estado y errores.
5. Modularización, dependencias y middlewares.
6. Headers, cookies y plantillas.
7. Autenticación y autorización con OAuth2.
8. Conexión a bases de datos con SQLModel.
9. Testing y checklist final.
```

Convenciones usadas en los ejemplos:

```text
Python:      3.10+
Pydantic:    v2
FastAPI:     estilo con Annotated cuando aporta claridad
Base de datos: PostgreSQL mediante SQLModel
```

---

# Parte I: fundamentos de diseño de APIs REST

## 1. Diseña rutas alrededor de recursos

Una API debe organizarse principalmente alrededor de **recursos**, no de acciones.

Correcto:

```http
GET /games
GET /games/16
GET /llm-models
GET /llm-models/2
```

Evita rutas con nombres de funciones:

```http
GET /get_games
GET /find_game
GET /return_model
```

El verbo ya está representado por el método HTTP:

```http
GET     /games       # leer juegos
POST    /games       # crear juego
GET     /games/16    # leer un juego específico
PATCH   /games/16    # actualizar parcialmente un juego
PUT     /games/16    # reemplazar un juego completo
DELETE  /games/16    # eliminar un juego
```

Regla práctica:

```text
La URL debe representar el recurso.
El método HTTP debe representar la acción.
```

---

## 2. Usa plural para colecciones

Una colección debería ir en plural:

```http
GET /games
GET /users
GET /books
GET /llm-models
```

Un recurso específico se representa agregando su identificador:

```http
GET /games/16
GET /users/42
GET /books/7
```

Evita mezclar singular y plural sin criterio:

```http
GET /game
GET /games
GET /game/16
```

---

## 3. Mantén nombres consistentes

Para URLs, usa nombres en minúsculas y separados por guiones:

```http
/games
/llm-models
/game-systems
```

Evita mezclar estilos:

```http
/llm_models
/llmModels
/LLMModels
```

En Python, usa `snake_case`:

```python
llm_models = []
get_llm_models()
search_games()
```

Convención recomendada:

```text
URL:     /llm-models
Python:  get_llm_models
```

---

## 4. Declara rutas fijas antes que rutas dinámicas

FastAPI evalúa las rutas en el orden en que fueron declaradas. Por eso, una ruta dinámica como `/games/{game_id}` puede capturar accidentalmente una ruta fija como `/games/search` si aparece antes.

Correcto:

```python
@app.get("/games/search")
def search_games(query: str):
    ...

@app.get("/games/{game_id}")
def get_game(game_id: int):
    ...
```

Incorrecto:

```python
@app.get("/games/{game_id}")
def get_game(game_id: int):
    ...

@app.get("/games/search")
def search_games(query: str):
    ...
```

En el caso incorrecto, FastAPI puede interpretar:

```http
/games/search
```

como:

```python
game_id = "search"
```

Si `game_id` está tipado como `int`, FastAPI intentará convertir `"search"` a entero y devolverá un error de validación.

Regla práctica:

```text
Rutas fijas primero.
Rutas dinámicas después.
```

---

## 5. Usa path parameters para identificar recursos concretos

Usa path parameters cuando el valor identifica una entidad concreta.

Correcto:

```http
GET /games/16
GET /llm-models/2
GET /users/42
```

Ejemplo:

```python
@app.get("/games/{game_id}")
def get_game(game_id: int):
    ...
```

FastAPI usa las anotaciones de tipo de Python para convertir y validar parámetros de ruta. Si declaras `game_id: int`, FastAPI intentará convertir el valor recibido a entero y devolverá un error si no puede hacerlo.

Evita usar path parameters para búsquedas flexibles:

```http
GET /games/Daggerheart
GET /games/FASA
GET /games/Medium
```

Eso genera ambigüedad. Es mejor usar query parameters.

---

## 6. Usa query parameters para filtros, búsquedas y paginación

Usa query parameters para buscar, filtrar, ordenar o paginar colecciones.

Correcto:

```http
GET /games?query=Daggerheart
GET /games?complexity=Medium
GET /games?company=FASA
GET /games?limit=10&offset=0
```

Ejemplo:

```python
@app.get("/games")
def get_games(
    query: str | None = None,
    complexity: str | None = None,
    company: str | None = None,
):
    results = games

    if query:
        results = [
            game for game in results
            if query.lower() in game["name"].lower()
        ]

    if complexity:
        results = [
            game for game in results
            if game["complexity"].lower() == complexity.lower()
        ]

    if company:
        results = [
            game for game in results
            if company.lower() in game["company"].lower()
        ]

    return results
```

Con esa estructura, no necesitas necesariamente un endpoint separado como:

```http
GET /games/search
```

Puedes usar directamente:

```http
GET /games?query=Daggerheart
```

---

## 7. No dupliques rutas con la misma forma

Evita declarar rutas que tengan la misma estructura:

```python
@app.get("/games/{game_id}")
def get_game_by_id(game_id: int):
    ...

@app.get("/games/{game_name}")
def get_game_by_name(game_name: str):
    ...
```

Aunque los nombres `game_id` y `game_name` sean distintos, la forma de la ruta es la misma:

```http
/games/algo
```

Mejor:

```python
@app.get("/games/{game_id}")
def get_game_by_id(game_id: int):
    ...

@app.get("/games/by-name/{game_name}")
def get_game_by_name(game_name: str):
    ...
```

O, preferiblemente para búsquedas:

```http
GET /games?name=Daggerheart
```

---

## 8. Usa `def` o `async def` con criterio

FastAPI permite usar tanto `def` como `async def`.

Usa `async def` cuando llames librerías asíncronas que requieren `await`:

```python
@app.get("/external-data")
async def get_external_data():
    result = await some_async_client.get_data()
    return result
```

Usa `def` cuando trabajes con código síncrono:

```python
@app.get("/games")
def get_games():
    return games
```

Regla práctica:

```text
Si usas await dentro del endpoint, usa async def.
Si no usas await y trabajas con librerías síncronas, usa def.
No mezcles una librería bloqueante dentro de async def sin entender el costo.
```

---

# Parte II: modelos y validación de datos

## 9. Usa modelos Pydantic para entradas y salidas

Evita manejar datos complejos solo como diccionarios sueltos. Define modelos.

```python
from pydantic import BaseModel

class Game(BaseModel):
    id: int
    name: str
    description: str
    complexity: str
    company: str
    ogl_url: str | None = None
```

Para crear recursos, usa un modelo sin `id` si el servidor debe generarlo:

```python
class GameCreate(BaseModel):
    name: str
    description: str
    complexity: str
    company: str
    ogl_url: str | None = None
```

Ejemplo:

```python
@app.post("/games", response_model=Game, status_code=201)
def create_game(game_data: GameCreate):
    new_game = {
        "id": len(games) + 1,
        **game_data.model_dump(),
    }

    games.append(new_game)

    return new_game
```

---

## 10. Separa modelos `Base`, `Create`, `Update` y `Read`

No siempre conviene usar el mismo modelo para crear, leer y actualizar.

```python
from pydantic import BaseModel, ConfigDict, Field

class GameBase(BaseModel):
    name: str = Field(min_length=2, max_length=120)
    description: str | None = Field(default=None, max_length=1_000)
    complexity: str = Field(pattern="^(Low|Medium|High)$")
    company: str = Field(min_length=2, max_length=120)
    ogl_url: str | None = None

class GameCreate(GameBase):
    pass

class GameUpdate(BaseModel):
    name: str | None = Field(default=None, min_length=2, max_length=120)
    description: str | None = Field(default=None, max_length=1_000)
    complexity: str | None = Field(default=None, pattern="^(Low|Medium|High)$")
    company: str | None = Field(default=None, min_length=2, max_length=120)
    ogl_url: str | None = None

class GameRead(GameBase):
    id: int
    model_config = ConfigDict(from_attributes=True)
```

Uso:

```python
@app.post("/games", response_model=GameRead, status_code=201)
def create_game(game_data: GameCreate):
    ...

@app.patch("/games/{game_id}", response_model=GameRead)
def update_game(game_id: int, game_data: GameUpdate):
    ...
```

Esto evita problemas como pedir `id` al crear un recurso o exigir todos los campos al hacer un `PATCH`.

---

## 11. Valida datos con `Field`

`Field` permite declarar restricciones y metadatos de campos.

```python
from pydantic import BaseModel, Field

class LLMModelCreate(BaseModel):
    name: str = Field(min_length=2, max_length=120, examples=["Gemma"])
    description: str = Field(min_length=10, max_length=1_000)
    size: str = Field(pattern=r"^\d+(B|M)$", examples=["7B"])
    url: str = Field(pattern=r"^https?://")
```

Reglas recomendadas:

```text
Valida longitud de strings.
Valida rangos numéricos.
Valida formatos conocidos.
Usa tipos específicos cuando existan.
No aceptes campos internos enviados por el cliente.
```

---

## 12. Usa tipos más expresivos cuando sea posible

Un tipo más expresivo reduce validaciones manuales.

```python
from enum import Enum
from pydantic import AnyUrl, BaseModel, EmailStr, Field

class Complexity(str, Enum):
    low = "Low"
    medium = "Medium"
    high = "High"

class GameCreate(BaseModel):
    name: str = Field(min_length=2, max_length=120)
    description: str | None = Field(default=None, max_length=1_000)
    complexity: Complexity
    company: str = Field(min_length=2, max_length=120)
    ogl_url: AnyUrl | None = None

class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(min_length=12, max_length=128)
```

Regla práctica:

```text
Prefiere Enum, EmailStr, AnyUrl, UUID, datetime y Decimal antes que str genérico.
```

---

## 13. Usa validadores cuando la regla no cabe en un tipo simple

Para reglas de negocio o normalización, usa validadores de Pydantic.

```python
from pydantic import BaseModel, Field, field_validator

class GameCreate(BaseModel):
    name: str = Field(min_length=2, max_length=120)
    company: str = Field(min_length=2, max_length=120)

    @field_validator("name", "company")
    @classmethod
    def strip_text(cls, value: str) -> str:
        value = value.strip()

        if not value:
            raise ValueError("Field cannot be empty")

        return value
```

Usa validadores para:

```text
Normalizar strings.
Validar reglas cruzadas.
Rechazar valores ambiguos.
Convertir formatos controlados.
```

Evita usarlos para lógica que necesita base de datos. Por ejemplo, validar que un nombre no exista debería hacerse en la capa de servicio o persistencia.

---

## 14. Usa `PATCH` para actualizaciones parciales

Usa `PATCH` cuando el cliente puede enviar solo algunos campos.

```python
from fastapi import HTTPException

@app.patch("/games/{game_id}", response_model=GameRead)
def update_game(game_id: int, game_data: GameUpdate):
    game = next((game for game in games if game["id"] == game_id), None)

    if game is None:
        raise HTTPException(status_code=404, detail="Game not found")

    update_data = game_data.model_dump(exclude_unset=True)

    for key, value in update_data.items():
        game[key] = value

    return game
```

Usa `PUT` solo cuando el cliente debe enviar una representación completa del recurso.

---

## 15. Evita `Body()` campo por campo para recursos complejos

Esta versión funciona:

```python
from fastapi import Body

@app.post("/llm-models")
def create_llm_model(
    id: int = Body(),
    name: str = Body(),
    description: str = Body(),
    size: str = Body(),
    url: str = Body(),
):
    ...
```

Pero obliga a declarar cada campo manualmente y hace que el endpoint crezca sin necesidad.

Mejor:

```python
from pydantic import BaseModel

class LLMModelCreate(BaseModel):
    name: str
    description: str
    size: str
    url: str

@app.post("/llm-models")
def create_llm_model(model_data: LLMModelCreate):
    ...
```

Regla práctica:

```text
Para uno o dos valores simples, Body() puede estar bien.
Para crear o actualizar recursos, usa modelos Pydantic.
```

---

# Parte III: validación de parámetros

## 16. Valida query parameters con `Query`

Para parámetros de búsqueda, filtros y paginación, usa `Query`.

```python
from typing import Annotated
from fastapi import Query

@app.get("/games", response_model=list[GameRead])
def get_games(
    query: Annotated[str | None, Query(min_length=2, max_length=50)] = None,
    complexity: Annotated[str | None, Query(pattern="^(Low|Medium|High)$")] = None,
    limit: Annotated[int, Query(ge=1, le=100)] = 10,
    offset: Annotated[int, Query(ge=0)] = 0,
):
    ...
```

Buenas prácticas:

```text
Usa límites máximos para paginación.
Valida strings de búsqueda con min_length y max_length.
Valida enums o patrones cuando el dominio es cerrado.
No permitas limit sin máximo en endpoints públicos.
```

---

## 17. Valida path parameters con `Path`

Para IDs y otros valores en ruta, usa `Path`.

```python
from typing import Annotated
from fastapi import Path

@app.get("/games/{game_id}", response_model=GameRead)
def get_game(
    game_id: Annotated[int, Path(ge=1, description="Game ID")],
):
    ...
```

También puedes validar rangos:

```python
@app.get("/reports/{year}")
def get_report(
    year: Annotated[int, Path(ge=2000, le=2100)],
):
    ...
```

Regla práctica:

```text
Un path parameter inválido debe fallar antes de llegar a tu lógica de negocio.
```

---

## 18. Valida headers con `Header`

Los headers son útiles para metadatos de request, trazabilidad, versiones internas o autenticación no estándar.

```python
from typing import Annotated
from fastapi import Header

@app.get("/health")
def health_check(
    user_agent: Annotated[str | None, Header()] = None,
    x_request_id: Annotated[str | None, Header(min_length=8, max_length=100)] = None,
):
    return {
        "status": "ok",
        "user_agent": user_agent,
        "x_request_id": x_request_id,
    }
```

FastAPI convierte por defecto guiones bajos en guiones. Por eso, `user_agent` lee el header `User-Agent`, y `x_request_id` lee `X-Request-Id`.

Reglas:

```text
Usa Header para metadatos HTTP.
No metas datos de negocio grandes en headers.
No uses headers personalizados para reemplazar OAuth2 o cookies sin razón.
```

---

## 19. Lee cookies con `Cookie`

Para leer cookies enviadas por el cliente:

```python
from typing import Annotated
from fastapi import Cookie

@app.get("/sessions/me")
def read_session(
    session_id: Annotated[str | None, Cookie()] = None,
):
    return {"session_id": session_id}
```

Regla práctica:

```text
Usa cookies para estado de sesión o tokens cuando el cliente sea navegador.
Usa Authorization: Bearer para APIs consumidas por clientes no navegador.
```

---

# Parte IV: respuestas, códigos de estado y errores

## 20. Usa `response_model`

Declara explícitamente qué forma tiene la respuesta.

```python
@app.get("/games", response_model=list[GameRead])
def get_games():
    return games

@app.get("/games/{game_id}", response_model=GameRead)
def get_game(game_id: int):
    ...
```

`response_model` ayuda a:

```text
Documentar la respuesta en OpenAPI.
Validar la estructura de salida.
Serializar datos a JSON.
Filtrar campos internos accidentalmente devueltos.
Facilitar generación de clientes.
```

Ejemplo típico para evitar filtrar contraseñas:

```python
class UserInDB(BaseModel):
    id: int
    email: str
    hashed_password: str

class UserRead(BaseModel):
    id: int
    email: str

@app.get("/users/{user_id}", response_model=UserRead)
def get_user(user_id: int):
    return UserInDB(
        id=user_id,
        email="user@example.com",
        hashed_password="not-for-clients",
    )
```

La respuesta no debería incluir `hashed_password`.

---

## 21. Usa códigos HTTP correctos

No devuelvas errores como si fueran respuestas exitosas.

Evita esto:

```python
if game is None:
    return {"error": "Game not found"}
```

Eso normalmente responde con `200 OK`, aunque el recurso no exista.

Mejor:

```python
from fastapi import HTTPException

@app.get("/games/{game_id}")
def get_game(game_id: int):
    game = next((game for game in games if game["id"] == game_id), None)

    if game is None:
        raise HTTPException(status_code=404, detail="Game not found")

    return game
```

Códigos comunes:

```text
200 OK                   # lectura exitosa
201 Created              # recurso creado
202 Accepted             # solicitud aceptada para procesamiento posterior
204 No Content            # operación exitosa sin cuerpo de respuesta
400 Bad Request           # solicitud inválida genérica
401 Unauthorized          # falta autenticación o token inválido
403 Forbidden             # autenticado, pero sin permisos
404 Not Found             # recurso no encontrado
409 Conflict              # duplicado o conflicto de estado
422 Unprocessable Entity  # validación fallida
500 Internal Server Error # error inesperado del servidor
```

Ejemplo:

```python
from fastapi import status

@app.post(
    "/games",
    response_model=GameRead,
    status_code=status.HTTP_201_CREATED,
)
def create_game(game_data: GameCreate):
    ...
```

---

## 22. Usa `204 No Content` sin body

Para eliminaciones exitosas, puedes usar `204 No Content`.

```python
from fastapi import Response, status

@app.delete("/games/{game_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_game(game_id: int):
    deleted = delete_game_by_id(game_id)

    if not deleted:
        raise HTTPException(status_code=404, detail="Game not found")

    return Response(status_code=status.HTTP_204_NO_CONTENT)
```

Regla:

```text
Si devuelves 204, no devuelvas cuerpo JSON.
```

---

## 23. Define tipos de respuesta cuando no devuelves JSON común

FastAPI usa JSON por defecto, pero puedes devolver otros tipos de respuesta.

### HTML

```python
from fastapi.responses import HTMLResponse

@app.get("/hello", response_class=HTMLResponse)
def hello():
    return "<h1>Hola</h1>"
```

### Redirección

```python
from fastapi.responses import RedirectResponse

@app.get("/docs-old")
def redirect_docs():
    return RedirectResponse(url="/docs")
```

### Archivo

```python
from fastapi.responses import FileResponse

@app.get("/reports/latest")
def latest_report():
    return FileResponse(
        path="reports/latest.pdf",
        media_type="application/pdf",
        filename="latest-report.pdf",
    )
```

### JSON personalizado

```python
from fastapi.responses import JSONResponse

@app.get("/custom")
def custom_response():
    return JSONResponse(
        status_code=200,
        content={"message": "Custom response"},
    )
```

Regla práctica:

```text
Usa response_model para datos de negocio.
Usa response_class o Response directa para HTML, archivos, redirects o respuestas especiales.
```

---

## 24. Documenta respuestas adicionales en OpenAPI

Cuando un endpoint puede devolver errores documentados, agrega `responses`.

```python
@app.get(
    "/games/{game_id}",
    response_model=GameRead,
    responses={
        404: {"description": "Game not found"},
        422: {"description": "Validation error"},
    },
)
def get_game(game_id: int):
    ...
```

Esto no reemplaza el manejo real de errores. Solo mejora la documentación.

---

## 25. Maneja errores con `HTTPException`

`HTTPException` debe levantarse, no retornarse.

```python
from fastapi import HTTPException, status

if game is None:
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail="Game not found",
    )
```

Puedes agregar headers en errores, por ejemplo en autenticación:

```python
raise HTTPException(
    status_code=status.HTTP_401_UNAUTHORIZED,
    detail="Invalid credentials",
    headers={"WWW-Authenticate": "Bearer"},
)
```

---

## 26. Crea errores personalizados cuando haya reglas de dominio

No todo error debe ser una cadena suelta. Para errores repetidos, crea una excepción de dominio y un handler.

```python
class GameAlreadyExistsError(Exception):
    def __init__(self, name: str):
        self.name = name

@app.exception_handler(GameAlreadyExistsError)
def game_already_exists_handler(request, exc: GameAlreadyExistsError):
    return JSONResponse(
        status_code=409,
        content={
            "error": "game_already_exists",
            "message": f"Game '{exc.name}' already exists",
        },
    )
```

Uso:

```python
@app.post("/games", response_model=GameRead, status_code=201)
def create_game(game_data: GameCreate):
    if game_name_exists(game_data.name):
        raise GameAlreadyExistsError(game_data.name)

    return save_game(game_data)
```

Ventajas:

```text
Separas lógica de dominio de transporte HTTP.
Evitas repetir respuestas de error.
Mantienes errores consistentes.
```

---

## 27. Personaliza errores de validación con cuidado

FastAPI ya devuelve errores detallados para validaciones fallidas. Si necesitas cambiar el formato global, usa un handler.

```python
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

@app.exception_handler(RequestValidationError)
def validation_exception_handler(request, exc: RequestValidationError):
    return JSONResponse(
        status_code=422,
        content={
            "error": "validation_error",
            "details": exc.errors(),
        },
    )
```

Regla práctica:

```text
No ocultes detalles útiles de validación durante desarrollo.
En producción, evita filtrar datos sensibles en errores.
```

---

## 28. Define una estructura de respuesta consistente

Evita que algunos endpoints devuelvan listas directas, otros diccionarios arbitrarios y otros strings si la API requiere consistencia estricta.

Para APIs pequeñas, esto puede estar bien:

```python
@app.get("/games", response_model=list[GameRead])
def get_games():
    return games
```

Para APIs donde quieres metadatos de paginación, usa un wrapper:

```python
from typing import Generic, TypeVar
from pydantic import BaseModel

T = TypeVar("T")

class Page(BaseModel, Generic[T]):
    items: list[T]
    total: int
    limit: int
    offset: int

@app.get("/games", response_model=Page[GameRead])
def get_games(limit: int = 10, offset: int = 0):
    return {
        "items": games[offset:offset + limit],
        "total": len(games),
        "limit": limit,
        "offset": offset,
    }
```

---

# Parte V: creación de recursos y errores frecuentes en Swagger

## 29. Crea recursos con `POST`

Para crear un recurso nuevo en FastAPI, usa el método `POST`.

Ejemplo menos recomendable, usando `Body()` campo por campo:

```python
from fastapi import Body

@app.post("/llm-models", tags=["LLM Models"])
def create_llm_model(
    id: int = Body(),
    name: str = Body(),
    description: str = Body(),
    size: str = Body(),
    url: str = Body(),
):
    llm_models.append({
        "id": id,
        "name": name,
        "description": description,
        "size": size,
        "url": url,
    })

    return {"message": "Model created successfully"}
```

Con esa definición, el body debe enviarse como objeto JSON:

```json
{
    "id": 4,
    "name": "Gemma",
    "description": "Modelo abierto de Google.",
    "size": "7B",
    "url": "https://huggingface.co/google/gemma-7b"
}
```

Importante: JSON no permite comas finales.

Incorrecto:

```json
{
    "id": 4,
    "name": "Gemma",
    "description": "Modelo abierto de Google.",
    "size": "7B",
    "url": "https://huggingface.co/google/gemma-7b",
}
```

Correcto:

```json
{
    "id": 4,
    "name": "Gemma",
    "description": "Modelo abierto de Google.",
    "size": "7B",
    "url": "https://huggingface.co/google/gemma-7b"
}
```

Ese error significa que el JSON está mal formado, no que la lógica del endpoint esté fallando.

---

## 30. Versión recomendada de `POST /llm-models`

```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel, Field

app = FastAPI(
    title="RPG Book Reaper",
    version="0.1.0",
)

llm_models = [
    {
        "id": 1,
        "name": "Llama",
        "description": "LLaMA is a collection of foundation language models.",
        "size": "7B",
        "url": "https://huggingface.co/meta-llama/Llama-2-7b-hf",
    },
    {
        "id": 2,
        "name": "Mistral",
        "description": "Mistral is a family of open-weight language models.",
        "size": "7B",
        "url": "https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.1",
    },
    {
        "id": 3,
        "name": "Falcon",
        "description": "Falcon is a family of open-weight language models.",
        "size": "7B",
        "url": "https://huggingface.co/tiiuae/falcon-7b-instruct",
    },
]

class LLMModelCreate(BaseModel):
    name: str = Field(min_length=2, max_length=120)
    description: str = Field(min_length=10, max_length=1_000)
    size: str = Field(pattern=r"^\d+(B|M)$")
    url: str = Field(pattern=r"^https?://")

class LLMModelRead(LLMModelCreate):
    id: int

@app.get("/llm-models", tags=["LLM Models"], response_model=list[LLMModelRead])
def get_llm_models():
    return llm_models

@app.post(
    "/llm-models",
    tags=["LLM Models"],
    response_model=LLMModelRead,
    status_code=status.HTTP_201_CREATED,
)
def create_llm_model(model_data: LLMModelCreate):
    duplicated = any(
        model["name"].lower() == model_data.name.lower()
        for model in llm_models
    )

    if duplicated:
        raise HTTPException(status_code=409, detail="Model already exists")

    new_id = max(model["id"] for model in llm_models) + 1

    new_model = {
        "id": new_id,
        **model_data.model_dump(),
    }

    llm_models.append(new_model)

    return new_model

@app.get("/llm-models/{model_id}", tags=["LLM Models"], response_model=LLMModelRead)
def get_llm_model(model_id: int):
    model = next((model for model in llm_models if model["id"] == model_id), None)

    if model is None:
        raise HTTPException(status_code=404, detail="Model not found")

    return model
```

Body para probar en Swagger:

```json
{
    "name": "Gemma",
    "description": "Modelo abierto de Google.",
    "size": "7B",
    "url": "https://huggingface.co/google/gemma-7b"
}
```

Después de ejecutarlo, puedes consultar:

```http
GET /llm-models
```

También puedes consultar directamente:

```http
GET /llm-models/4
```

---

## 31. Errores frecuentes al probar `POST` en Swagger

### Error 1: coma final en JSON

Incorrecto:

```json
{
    "name": "Gemma",
    "description": "Modelo abierto de Google.",
    "size": "7B",
    "url": "https://huggingface.co/google/gemma-7b",
}
```

Correcto:

```json
{
    "name": "Gemma",
    "description": "Modelo abierto de Google.",
    "size": "7B",
    "url": "https://huggingface.co/google/gemma-7b"
}
```

### Error 2: campo faltante

Si tu modelo exige `name`, `description`, `size` y `url`, este body fallará:

```json
{
    "name": "Gemma",
    "size": "7B"
}
```

Faltan:

```text
description
url
```

### Error 3: tipo incorrecto

Si declaras:

```python
id: int
```

este body fallará:

```json
{
    "id": "cuatro",
    "name": "Gemma",
    "description": "Modelo abierto de Google.",
    "size": "7B",
    "url": "https://huggingface.co/google/gemma-7b"
}
```

Porque `"cuatro"` no puede convertirse a entero.

### Error 4: enviar JSON como texto inválido

No uses comillas externas alrededor del objeto completo:

```json
"{ \"name\": \"Gemma\", \"size\": \"7B\" }"
```

Eso es un string, no un objeto JSON.

Debe ser:

```json
{
    "name": "Gemma",
    "description": "Modelo abierto de Google.",
    "size": "7B",
    "url": "https://huggingface.co/google/gemma-7b"
}
```

---

# Parte VI: modularización y arquitectura interna

## 32. Agrupa endpoints con `APIRouter`

Cuando el proyecto crece, evita poner todo en `main.py`. Usa `APIRouter`.

Estructura sugerida:

```text
api/
    src/
        main.py
        core/
            config.py
            security.py
        db/
            session.py
        models/
            game.py
            user.py
        routers/
            games.py
            llm_models.py
            auth.py
        schemas/
            game.py
            user.py
        services/
            game_service.py
            user_service.py
        exceptions.py
```

Ejemplo en `routers/games.py`:

```python
from fastapi import APIRouter, HTTPException

router = APIRouter(
    prefix="/games",
    tags=["Games"],
)

@router.get("")
def get_games():
    return games

@router.get("/{game_id}")
def get_game(game_id: int):
    game = next((game for game in games if game["id"] == game_id), None)

    if game is None:
        raise HTTPException(status_code=404, detail="Game not found")

    return game
```

Ejemplo en `main.py`:

```python
from fastapi import FastAPI
from api.src.routers import games, llm_models

app = FastAPI(
    title="RPG Book Reaper",
    version="0.1.0",
)

app.include_router(games.router)
app.include_router(llm_models.router)
```

---

## 33. No mezcles lógica de negocio con endpoints

El endpoint debería encargarse de:

```text
Recibir parámetros.
Validar datos de entrada.
Llamar a una función de servicio.
Devolver una respuesta HTTP.
```

Evita que el endpoint tenga demasiada lógica.

Menos recomendable:

```python
@app.get("/games/{game_id}")
def get_game(game_id: int):
    game = None

    for item in games:
        if item["id"] == game_id:
            game = item
            break

    if game is None:
        raise HTTPException(status_code=404, detail="Game not found")

    return game
```

Mejor:

```python
def find_game_by_id(game_id: int) -> dict | None:
    return next((game for game in games if game["id"] == game_id), None)

@app.get("/games/{game_id}")
def get_game(game_id: int):
    game = find_game_by_id(game_id)

    if game is None:
        raise HTTPException(status_code=404, detail="Game not found")

    return game
```

En proyectos medianos, mueve `find_game_by_id()` a `services/game_service.py`.

---

## 34. Usa tags, summary y description para mejorar la documentación

FastAPI genera documentación automática. Puedes mejorarla agregando metadatos a tus endpoints.

```python
@app.get(
    "/games",
    tags=["Games"],
    summary="List games",
    description="Returns the list of available tabletop RPG games.",
)
def get_games():
    return games
```

También puedes configurar metadatos generales de la app:

```python
app = FastAPI(
    title="RPG Book Reaper",
    version="0.1.0",
    description="API for listing tabletop RPG games and LLM models.",
)
```

---

## 35. Usa configuración centralizada

Evita hardcodear secretos, URLs y flags de entorno.

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    app_name: str = "RPG Book Reaper"
    environment: str = "local"
    database_url: str
    secret_key: str
    access_token_expire_minutes: int = 30

    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8")

settings = Settings()
```

Ejemplo `.env`:

```dotenv
DATABASE_URL=postgresql+psycopg://app_user:change_me@localhost:5432/rpg_book_reaper
SECRET_KEY=change-this-in-production
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

Reglas:

```text
No guardes secretos reales en Git.
Usa variables de entorno en producción.
Usa valores distintos por ambiente.
```

---

# Parte VII: dependencias

## 36. Usa `Depends` para recursos compartidos

FastAPI tiene un sistema de inyección de dependencias. Sirve para compartir lógica entre endpoints sin repetir código.

Ejemplo simple para paginación:

```python
from typing import Annotated
from fastapi import Depends, Query
from pydantic import BaseModel

class PaginationParams(BaseModel):
    limit: int
    offset: int

def get_pagination(
    limit: Annotated[int, Query(ge=1, le=100)] = 10,
    offset: Annotated[int, Query(ge=0)] = 0,
) -> PaginationParams:
    return PaginationParams(limit=limit, offset=offset)

@app.get("/games")
def get_games(
    pagination: Annotated[PaginationParams, Depends(get_pagination)],
):
    return games[pagination.offset:pagination.offset + pagination.limit]
```

Usa dependencias para:

```text
Sesiones de base de datos.
Usuario autenticado.
Verificación de permisos.
Paginación común.
Filtros comunes.
Clientes externos.
Configuración.
```

---

## 37. Usa dependencias con `yield` para abrir y cerrar recursos

Las dependencias con `yield` son útiles para recursos que deben cerrarse después del request, como una sesión de base de datos.

```python
from typing import Generator
from sqlmodel import Session

from api.src.db.session import engine

def get_session() -> Generator[Session, None, None]:
    with Session(engine) as session:
        yield session
```

Uso:

```python
from typing import Annotated
from fastapi import Depends
from sqlmodel import Session

SessionDep = Annotated[Session, Depends(get_session)]

@app.get("/games")
def get_games(session: SessionDep):
    ...
```

Regla práctica:

```text
La dependencia abre el recurso.
El endpoint lo usa.
La dependencia lo cierra.
```

---

## 38. Usa dependencias a nivel de router o aplicación

Si una validación aplica a muchos endpoints, no la repitas.

```python
from typing import Annotated
from fastapi import APIRouter, Depends, Header, HTTPException

async def verify_internal_token(
    x_internal_token: Annotated[str, Header()],
):
    if x_internal_token != "expected-token":
        raise HTTPException(status_code=403, detail="Invalid internal token")

router = APIRouter(
    prefix="/admin",
    tags=["Admin"],
    dependencies=[Depends(verify_internal_token)],
)
```

Regla práctica:

```text
Usa dependencias globales solo para reglas realmente globales.
Usa dependencias por router para permisos o precondiciones por área.
```

---

# Parte VIII: middlewares y manejo transversal

## 39. Usa middlewares para lógica transversal

Un middleware se ejecuta antes y después de cada request. Úsalo para lógica transversal, no para lógica de negocio.

Ejemplo: medir duración de request.

```python
import time
from fastapi import Request

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.perf_counter()
    response = await call_next(request)
    process_time = time.perf_counter() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

Buenos usos:

```text
Logging de requests.
Request ID.
Medición de tiempos.
Headers de seguridad.
CORS.
Compresión.
```

Malos usos:

```text
Validar reglas de negocio específicas.
Abrir transacciones complejas sin control.
Modificar silenciosamente cuerpos de request.
Hacer queries pesadas en cada request.
```

---

## 40. Configura CORS si un frontend consumirá tu API

Si una aplicación frontend en otro origen necesita llamar a tu API, configura CORS explícitamente.

```python
from fastapi.middleware.cors import CORSMiddleware

origins = [
    "http://localhost:3000",
    "http://localhost:5173",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

Evita esto en producción si tu API maneja autenticación, cookies o datos sensibles:

```python
allow_origins=["*"]
```

Reglas:

```text
Lista explícitamente dominios permitidos.
No uses wildcard con credenciales.
Configura cookies con SameSite y Secure cuando corresponda.
```

---

## 41. Agrega headers de seguridad básicos

Puedes agregar headers de seguridad con middleware.

```python
@app.middleware("http")
async def security_headers(request: Request, call_next):
    response = await call_next(request)
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
    return response
```

Regla práctica:

```text
Para APIs JSON puras, prioriza CORS correcto, HTTPS y autenticación robusta.
Para respuestas HTML, revisa también CSP, cookies y protección contra CSRF.
```

---

# Parte IX: headers y cookies

## 42. Crea cookies con `Response.set_cookie`

Para crear una cookie, declara un parámetro `Response` y llama `set_cookie()`.

```python
from fastapi import Response

@app.post("/sessions")
def create_session(response: Response):
    response.set_cookie(
        key="session_id",
        value="abc123",
        max_age=60 * 60,
        httponly=True,
        secure=True,
        samesite="lax",
    )

    return {"message": "Session created"}
```

Parámetros importantes:

```text
httponly=True    # evita acceso desde JavaScript
secure=True      # envía cookie solo por HTTPS
samesite="lax"   # reduce riesgo CSRF en navegación normal
max_age=3600     # duración en segundos
path="/"         # alcance de la cookie
```

---

## 43. Usa cookies HttpOnly para tokens sensibles en navegador

Si guardas tokens en cookies para un frontend web, usa `httponly=True`.

```python
@app.post("/auth/login")
def login(response: Response):
    access_token = create_access_token(subject="user@example.com")

    response.set_cookie(
        key="access_token",
        value=access_token,
        httponly=True,
        secure=True,
        samesite="lax",
        max_age=60 * 30,
    )

    return {"message": "Logged in"}
```

Para borrar una cookie:

```python
@app.post("/auth/logout")
def logout(response: Response):
    response.delete_cookie(key="access_token")
    return {"message": "Logged out"}
```

Advertencia:

```text
HttpOnly reduce exposición ante XSS, pero no elimina necesidad de proteger contra CSRF si usas cookies automáticamente enviadas por el navegador.
```

---

## 44. Decide entre Bearer token y cookie según el cliente

```text
Authorization: Bearer <token>
- Bueno para APIs consumidas por apps móviles, CLI, servicios backend o SPAs que gestionan tokens explícitamente.
- El cliente debe enviar el header manualmente.

Cookie HttpOnly
- Buena para aplicaciones web navegador-servidor.
- El navegador envía la cookie automáticamente.
- Requiere revisar CSRF, SameSite, Secure y CORS.
```

Regla práctica:

```text
No mezcles ambos enfoques sin una razón clara.
Documenta dónde vive el token y cómo se renueva.
```

---

# Parte X: motores de plantillas

## 45. Usa Jinja2Templates para HTML server-side

FastAPI puede devolver HTML usando motores de plantillas. La opción común es Jinja2.

Instalación:

```bash
pip install jinja2
```

Estructura:

```text
api/
    src/
        main.py
        templates/
            item.html
        static/
            styles.css
```

Ejemplo:

```python
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

app = FastAPI()

app.mount("/static", StaticFiles(directory="static"), name="static")

templates = Jinja2Templates(directory="templates")

@app.get("/items/{item_id}", response_class=HTMLResponse)
async def read_item(request: Request, item_id: str):
    return templates.TemplateResponse(
        request=request,
        name="item.html",
        context={"item_id": item_id},
    )
```

Template `templates/item.html`:

```html
<html>
<head>
    <title>Item Details</title>
    <link href="{{ url_for('static', path='/styles.css') }}" rel="stylesheet">
</head>
<body>
    <h1>Item ID: {{ item_id }}</h1>
</body>
</html>
```

Reglas:

```text
Usa templates para HTML server-side.
Usa JSON para APIs consumidas por frontends separados.
No mezcles HTML y JSON arbitrariamente en los mismos endpoints.
```

---

# Parte XI: autenticación y autorización con OAuth2

## 46. Distingue autenticación y autorización

```text
Autenticación: comprobar quién es el usuario.
Autorización: comprobar qué puede hacer ese usuario.
```

Ejemplo:

```text
Usuario autenticado: felipe@example.com
Rol: admin
Permiso: create:game
```

Una API debería responder:

```text
401 Unauthorized  # no sé quién eres o el token es inválido
403 Forbidden     # sé quién eres, pero no puedes hacer esto
```

---

## 47. Usa OAuth2PasswordBearer para recibir Bearer tokens

```python
from typing import Annotated
from fastapi import Depends
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/token")

@app.get("/debug-token")
def debug_token(token: Annotated[str, Depends(oauth2_scheme)]):
    return {"token": token}
```

El cliente envía:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

---

## 48. Implementa login con OAuth2 Password Flow y JWT

Instalación sugerida:

```bash
pip install "python-multipart" "pyjwt[crypto]" "pwdlib[bcrypt]"
```

Ejemplo base:

```python
from datetime import datetime, timedelta, timezone
from typing import Annotated

import jwt
from jwt import InvalidTokenError
from pwdlib import PasswordHash

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel

SECRET_KEY = "change-this-secret-in-production"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

password_hash = PasswordHash.recommended()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/token")

app = FastAPI()

fake_users_db = {
    "felipe": {
        "username": "felipe",
        "full_name": "Felipe",
        "email": "felipe@example.com",
        "hashed_password": password_hash.hash("not-a-real-password"),
        "disabled": False,
        "roles": ["admin"],
    }
}

class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: str | None = None

class User(BaseModel):
    username: str
    email: str | None = None
    full_name: str | None = None
    disabled: bool = False
    roles: list[str] = []

class UserInDB(User):
    hashed_password: str

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return password_hash.verify(plain_password, hashed_password)

def get_user(username: str) -> UserInDB | None:
    user_dict = fake_users_db.get(username)

    if not user_dict:
        return None

    return UserInDB(**user_dict)

def authenticate_user(username: str, password: str) -> UserInDB | None:
    user = get_user(username)

    if not user:
        return None

    if not verify_password(password, user.hashed_password):
        return None

    return user

def create_access_token(data: dict, expires_delta: timedelta | None = None) -> str:
    to_encode = data.copy()
    expire = datetime.now(timezone.utc) + (
        expires_delta or timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    )
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

async def get_current_user(
    token: Annotated[str, Depends(oauth2_scheme)],
) -> User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )

    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username = payload.get("sub")

        if username is None:
            raise credentials_exception

        token_data = TokenData(username=username)
    except InvalidTokenError:
        raise credentials_exception

    user = get_user(username=token_data.username)

    if user is None:
        raise credentials_exception

    return user

async def get_current_active_user(
    current_user: Annotated[User, Depends(get_current_user)],
) -> User:
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")

    return current_user

@app.post("/auth/token", response_model=Token)
async def login_for_access_token(
    form_data: Annotated[OAuth2PasswordRequestForm, Depends()],
) -> Token:
    user = authenticate_user(form_data.username, form_data.password)

    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )

    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username},
        expires_delta=access_token_expires,
    )

    return Token(access_token=access_token, token_type="bearer")

@app.get("/users/me", response_model=User)
async def read_users_me(
    current_user: Annotated[User, Depends(get_current_active_user)],
):
    return current_user
```

Reglas mínimas:

```text
No guardes contraseñas en texto plano.
No hardcodees SECRET_KEY en producción.
Usa HTTPS.
Define expiración de tokens.
No devuelvas hashed_password en response_model.
```

---

## 49. Aplica autorización por roles o permisos

Ejemplo de dependencia para exigir rol `admin`:

```python
from typing import Annotated
from fastapi import Depends, HTTPException, status

async def require_admin(
    current_user: Annotated[User, Depends(get_current_active_user)],
) -> User:
    if "admin" not in current_user.roles:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Not enough permissions",
        )

    return current_user

@app.post("/games", response_model=GameRead, status_code=201)
def create_game(
    game_data: GameCreate,
    current_user: Annotated[User, Depends(require_admin)],
):
    return save_game(game_data)
```

Para proyectos más grandes, usa permisos específicos:

```text
read:games
create:games
update:games
delete:games
admin:users
```

---

## 50. Usa scopes de OAuth2 cuando necesites permisos granulares

FastAPI soporta scopes en OAuth2. Conceptualmente:

```text
Token A: scopes = ["games:read"]
Token B: scopes = ["games:read", "games:write"]
```

Ejemplo simplificado:

```python
from fastapi import Security
from fastapi.security import SecurityScopes

oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="/auth/token",
    scopes={
        "games:read": "Read games",
        "games:write": "Create and update games",
    },
)

async def get_current_user_with_scopes(
    security_scopes: SecurityScopes,
    token: Annotated[str, Depends(oauth2_scheme)],
) -> User:
    user = await get_current_user(token)

    token_scopes = decode_scopes_from_token(token)

    for scope in security_scopes.scopes:
        if scope not in token_scopes:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail="Not enough permissions",
            )

    return user

@app.post("/games")
def create_game(
    game_data: GameCreate,
    current_user: Annotated[
        User,
        Security(get_current_user_with_scopes, scopes=["games:write"]),
    ],
):
    ...
```

Regla práctica:

```text
Roles sirven para reglas simples.
Scopes o permisos sirven para sistemas más granulares.
```

---

# Parte XII: conexión a bases de datos con SQLModel

## 51. Instala dependencias para SQLModel y PostgreSQL

```bash
pip install sqlmodel "psycopg[binary]"
```

Para configuración con `.env`:

```bash
pip install pydantic-settings
```

Ejemplo de URL de conexión:

```text
postgresql+psycopg://app_user:change_me@localhost:5432/rpg_book_reaper
```

Reglas:

```text
No uses el superusuario de PostgreSQL desde la app.
No guardes contraseñas en el repositorio.
Usa un rol con permisos mínimos necesarios.
```

---

## 52. Define modelos SQLModel

`SQLModel` puede representar tablas y esquemas de validación. Para proyectos simples, puedes usar modelos relacionados; para proyectos medianos, separa modelos de tabla y modelos de API.

```python
from sqlmodel import Field, SQLModel

class GameBase(SQLModel):
    name: str = Field(index=True, min_length=2, max_length=120)
    description: str | None = Field(default=None, max_length=1_000)
    complexity: str = Field(index=True, max_length=20)
    company: str = Field(index=True, min_length=2, max_length=120)
    ogl_url: str | None = None

class Game(GameBase, table=True):
    __tablename__ = "games"

    id: int | None = Field(default=None, primary_key=True)

class GameCreate(GameBase):
    pass

class GameRead(GameBase):
    id: int

class GameUpdate(SQLModel):
    name: str | None = Field(default=None, min_length=2, max_length=120)
    description: str | None = Field(default=None, max_length=1_000)
    complexity: str | None = Field(default=None, max_length=20)
    company: str | None = Field(default=None, min_length=2, max_length=120)
    ogl_url: str | None = None
```

Notas:

```text
table=True indica que el modelo representa una tabla.
id puede ser None antes de persistir el objeto.
Field(primary_key=True) marca la clave primaria.
Field(index=True) crea un índice si usas SQLModel.metadata.create_all().
```

Para producción, usa migraciones con Alembic en vez de depender solo de `create_all()`.

---

## 53. Crea el engine y la sesión

Archivo `db/session.py`:

```python
from sqlmodel import Session, SQLModel, create_engine

from api.src.core.config import settings

engine = create_engine(
    settings.database_url,
    echo=settings.environment == "local",
    pool_pre_ping=True,
)

def create_db_and_tables() -> None:
    SQLModel.metadata.create_all(engine)

def get_session():
    with Session(engine) as session:
        yield session
```

Archivo `main.py`:

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

from api.src.db.session import create_db_and_tables
from api.src.routers import games

@asynccontextmanager
async def lifespan(app: FastAPI):
    create_db_and_tables()
    yield

app = FastAPI(
    title="RPG Book Reaper",
    version="0.1.0",
    lifespan=lifespan,
)

app.include_router(games.router)
```

Advertencia:

```text
create_all() es cómodo en desarrollo.
En producción, usa migraciones versionadas.
```

---

## 54. Inyecta la sesión con `Depends`

```python
from typing import Annotated
from fastapi import Depends
from sqlmodel import Session

from api.src.db.session import get_session

SessionDep = Annotated[Session, Depends(get_session)]
```

Ahora puedes usar `SessionDep` en routers:

```python
@router.get("/games")
def list_games(session: SessionDep):
    ...
```

---

## 55. Crea endpoints CRUD con SQLModel

Archivo `routers/games.py`:

```python
from typing import Annotated

from fastapi import APIRouter, Depends, HTTPException, Query, status
from sqlmodel import Session, select

from api.src.db.session import get_session
from api.src.models.game import Game, GameCreate, GameRead, GameUpdate

router = APIRouter(prefix="/games", tags=["Games"])

SessionDep = Annotated[Session, Depends(get_session)]

@router.post(
    "",
    response_model=GameRead,
    status_code=status.HTTP_201_CREATED,
)
def create_game(game_data: GameCreate, session: SessionDep):
    existing = session.exec(
        select(Game).where(Game.name == game_data.name)
    ).first()

    if existing:
        raise HTTPException(status_code=409, detail="Game already exists")

    game = Game.model_validate(game_data)
    session.add(game)
    session.commit()
    session.refresh(game)

    return game

@router.get("", response_model=list[GameRead])
def list_games(
    session: SessionDep,
    query: Annotated[str | None, Query(min_length=2, max_length=50)] = None,
    limit: Annotated[int, Query(ge=1, le=100)] = 20,
    offset: Annotated[int, Query(ge=0)] = 0,
):
    statement = select(Game)

    if query:
        statement = statement.where(Game.name.ilike(f"%{query}%"))

    statement = statement.offset(offset).limit(limit)

    return session.exec(statement).all()

@router.get("/{game_id}", response_model=GameRead)
def get_game(game_id: int, session: SessionDep):
    game = session.get(Game, game_id)

    if not game:
        raise HTTPException(status_code=404, detail="Game not found")

    return game

@router.patch("/{game_id}", response_model=GameRead)
def update_game(game_id: int, game_data: GameUpdate, session: SessionDep):
    game = session.get(Game, game_id)

    if not game:
        raise HTTPException(status_code=404, detail="Game not found")

    update_data = game_data.model_dump(exclude_unset=True)

    for key, value in update_data.items():
        setattr(game, key, value)

    session.add(game)
    session.commit()
    session.refresh(game)

    return game

@router.delete("/{game_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_game(game_id: int, session: SessionDep):
    game = session.get(Game, game_id)

    if not game:
        raise HTTPException(status_code=404, detail="Game not found")

    session.delete(game)
    session.commit()
```

Reglas:

```text
Usa session.get(Model, id) para búsqueda por primary key.
Usa select(Model).where(...) para consultas filtradas.
Haz commit después de crear, actualizar o eliminar.
Haz refresh después de crear o actualizar si necesitas datos generados por la base.
```

---

## 56. Evita SQL injection usando el ORM correctamente

Evita construir SQL con interpolación de strings:

```python
query = f"SELECT * FROM games WHERE name = '{name}'"
```

Mejor con SQLModel:

```python
statement = select(Game).where(Game.name == name)
game = session.exec(statement).first()
```

Para búsquedas parciales:

```python
statement = select(Game).where(Game.name.ilike(f"%{query}%"))
```

Aunque el patrón incluya interpolación para armar el valor, no estás concatenando SQL manual. El ORM genera una consulta parametrizada.

---

## 57. Maneja errores de base de datos

Los errores de integridad deberían transformarse en errores HTTP claros.

```python
from fastapi import HTTPException
from sqlalchemy.exc import IntegrityError

try:
    session.add(game)
    session.commit()
except IntegrityError:
    session.rollback()
    raise HTTPException(status_code=409, detail="Database integrity conflict")
```

Reglas:

```text
Haz rollback cuando falle una transacción.
No expongas mensajes internos de la base al cliente.
Registra el error real en logs internos.
Usa constraints en base de datos además de validaciones en API.
```

---

## 58. Usa migraciones para producción

`SQLModel.metadata.create_all(engine)` sirve para desarrollo o prototipos. Para producción, usa migraciones.

Herramienta habitual:

```text
Alembic
```

Buenas prácticas:

```text
Versiona cambios de esquema.
Revisa SQL generado antes de aplicarlo.
Prueba migraciones en staging.
Ten plan de rollback.
No apliques cambios manuales en producción sin registro.
```

---

## 59. Controla transacciones explícitamente cuando sea necesario

Para una operación simple, este patrón suele bastar:

```python
session.add(game)
session.commit()
session.refresh(game)
```

Para varias operaciones relacionadas:

```python
try:
    session.add(game)
    session.add(audit_record)
    session.commit()
except Exception:
    session.rollback()
    raise
```

Regla:

```text
Si una operación lógica modifica varias tablas, debe confirmarse o revertirse completa.
```

---

# Parte XIII: ejemplo integral mínimo

## 60. `main.py`

```python
from contextlib import asynccontextmanager

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from api.src.db.session import create_db_and_tables
from api.src.routers import auth, games, llm_models

@asynccontextmanager
async def lifespan(app: FastAPI):
    create_db_and_tables()
    yield

app = FastAPI(
    title="RPG Book Reaper",
    version="0.1.0",
    description="API for tabletop RPG games and LLM models.",
    lifespan=lifespan,
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(auth.router)
app.include_router(games.router)
app.include_router(llm_models.router)

@app.get("/", tags=["Home"])
def home():
    return {
        "message": "Hola Mundo",
        "app": "RPG Book Reaper",
        "version": "0.1.0",
    }
```

---

## 61. Checklist rápida antes de agregar un endpoint

```text
[ ] ¿La ruta representa un recurso?
[ ] ¿El método HTTP corresponde a la acción?
[ ] ¿Las rutas fijas están antes que las dinámicas?
[ ] ¿Los IDs van como path parameters?
[ ] ¿Los filtros van como query parameters?
[ ] ¿La ruta usa nombres consistentes?
[ ] ¿La entrada tiene modelo Pydantic o SQLModel adecuado?
[ ] ¿La respuesta tiene response_model?
[ ] ¿Los errores usan HTTPException o handler personalizado?
[ ] ¿Los códigos HTTP son correctos?
[ ] ¿La colección tiene paginación si puede crecer?
[ ] ¿El endpoint no mezcla demasiada lógica de negocio?
[ ] ¿La sesión de base de datos entra por Depends?
[ ] ¿Los permisos están declarados como dependencias?
[ ] ¿Hay tests para casos exitosos y fallidos?
```

---

# Parte XIV: testing

## 62. Escribe tests para tus endpoints

FastAPI incluye soporte para probar endpoints usando `TestClient`.

```python
from fastapi.testclient import TestClient
from api.src.main import app

client = TestClient(app)

def test_get_games():
    response = client.get("/games")

    assert response.status_code == 200
    assert isinstance(response.json(), list)

def test_get_game_not_found():
    response = client.get("/games/999")

    assert response.status_code == 404
    assert response.json() == {"detail": "Game not found"}
```

Prueba como mínimo:

```text
Endpoints exitosos.
Recursos inexistentes.
Parámetros inválidos.
Filtros y búsquedas.
Creación y actualización de recursos.
Duplicados y conflictos.
Autenticación inválida.
Permisos insuficientes.
Respuestas esperadas.
```

---

## 63. Sobrescribe dependencias en tests

Para tests con base de datos temporal o sesión distinta, sobrescribe dependencias.

```python
from fastapi.testclient import TestClient
from sqlmodel import Session, SQLModel, create_engine
from sqlmodel.pool import StaticPool

from api.src.db.session import get_session
from api.src.main import app

engine = create_engine(
    "sqlite://",
    connect_args={"check_same_thread": False},
    poolclass=StaticPool,
)

SQLModel.metadata.create_all(engine)

def get_test_session():
    with Session(engine) as session:
        yield session

app.dependency_overrides[get_session] = get_test_session

client = TestClient(app)

def test_create_game():
    response = client.post(
        "/games",
        json={
            "name": "Daggerheart",
            "description": "Fantasy RPG.",
            "complexity": "Medium",
            "company": "Darrington Press",
            "ogl_url": None,
        },
    )

    assert response.status_code == 201
    assert response.json()["name"] == "Daggerheart"
```

Regla:

```text
Los tests no deben depender de la base de datos real de desarrollo o producción.
```

---

# Parte XV: resumen de reglas principales

```text
1. Rutas fijas antes que rutas dinámicas.
2. Recursos en plural: /games, /users, /books.
3. IDs concretos como path params: /games/16.
4. Filtros y búsquedas como query params: /games?query=Daggerheart.
5. Usa Pydantic o SQLModel para validar entrada y salida.
6. Separa modelos Create, Read y Update.
7. Usa response_model para documentar y filtrar respuestas.
8. Usa HTTPException para errores HTTP.
9. Usa códigos de estado correctos.
10. Usa APIRouter cuando el proyecto crezca.
11. Separa routers, schemas, models, services, db y core.
12. Usa Depends para sesiones, usuario actual y permisos.
13. Usa middleware solo para lógica transversal.
14. Usa cookies HttpOnly si guardas tokens sensibles en navegador.
15. Usa OAuth2/JWT con contraseñas hasheadas y expiración.
16. Usa SQLModel con sesiones por request.
17. Usa migraciones para producción.
18. Escribe tests desde el inicio.
```

---

# Referencias oficiales recomendadas

```text
FastAPI - Tutorial User Guide:
https://fastapi.tiangolo.com/tutorial/

FastAPI - Request Body:
https://fastapi.tiangolo.com/tutorial/body/

FastAPI - Query Parameters and String Validations:
https://fastapi.tiangolo.com/tutorial/query-params-str-validations/

FastAPI - Path Parameters and Numeric Validations:
https://fastapi.tiangolo.com/tutorial/path-params-numeric-validations/

FastAPI - Header Parameters:
https://fastapi.tiangolo.com/tutorial/header-params/

FastAPI - Cookie Parameters:
https://fastapi.tiangolo.com/tutorial/cookie-params/

FastAPI - Response Model:
https://fastapi.tiangolo.com/tutorial/response-model/

FastAPI - Response Status Code:
https://fastapi.tiangolo.com/tutorial/response-status-code/

FastAPI - Handling Errors:
https://fastapi.tiangolo.com/tutorial/handling-errors/

FastAPI - Dependencies:
https://fastapi.tiangolo.com/tutorial/dependencies/

FastAPI - Middleware:
https://fastapi.tiangolo.com/tutorial/middleware/

FastAPI - Bigger Applications:
https://fastapi.tiangolo.com/tutorial/bigger-applications/

FastAPI - Templates:
https://fastapi.tiangolo.com/advanced/templates/

FastAPI - Response Cookies:
https://fastapi.tiangolo.com/advanced/response-cookies/

FastAPI - OAuth2 with Password and JWT:
https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/

SQLModel - FastAPI tutorial:
https://sqlmodel.tiangolo.com/tutorial/fastapi/

SQLModel - Session with FastAPI Dependency:
https://sqlmodel.tiangolo.com/tutorial/fastapi/session-with-dependency/
```
