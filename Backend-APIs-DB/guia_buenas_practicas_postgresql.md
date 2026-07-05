# Guía de buenas prácticas para creación, uso y mantención de bases de datos SQL con énfasis en PostgreSQL

## 1. Objetivo

Esta guía resume buenas prácticas para trabajar con bases de datos SQL, con énfasis en PostgreSQL. Está organizada en tres niveles:

```text id="8qa9bl"
1. Básico: creación, tablas, consultas, relaciones y operaciones CRUD.
2. Intermedio: diseño, índices, migraciones, seguridad, backups y rendimiento inicial.
3. Avanzado: particionamiento, replicación, tuning, observabilidad, alta disponibilidad y mantenimiento en producción.
```

PostgreSQL es un sistema de base de datos objeto-relacional de código abierto, con soporte amplio del estándar SQL y varias características modernas. La documentación actual estable de PostgreSQL corresponde a la rama 18; PostgreSQL 19 está disponible como beta, por lo que no debería asumirse como versión de producción salvo decisión técnica explícita.

---

# Parte I: fundamentos básicos

## 2. Qué es una base de datos SQL

Una base de datos SQL organiza información en tablas relacionadas entre sí.

Una tabla contiene:

```text id="6pfioa"
Filas     = registros
Columnas  = atributos
```

Ejemplo de tabla `games`:

```text id="efnb83"
id | name                  | complexity | company
---|-----------------------|------------|----------------------
1  | Dungeons & Dragons    | Low        | Wizards of the Coast
2  | Call of Cthulhu       | Medium     | Chaosium
```

SQL permite:

```text id="y6p6ay"
CREATE   # crear estructuras
INSERT   # insertar datos
SELECT   # consultar datos
UPDATE   # modificar datos
DELETE   # eliminar datos
```

---

## 3. Conceptos básicos de PostgreSQL

En PostgreSQL conviene distinguir estos conceptos:

```text id="qaad1u"
Cluster     # instancia completa de PostgreSQL con una o más bases de datos
Database    # base de datos concreta dentro del cluster
Schema      # espacio lógico dentro de una base de datos
Table       # tabla de datos
Row         # fila o registro
Column      # columna o campo
Role        # usuario o grupo de permisos
Index       # estructura auxiliar para acelerar consultas
Constraint  # regla de integridad
```

Ejemplo:

```text id="lqyo5v"
Cluster PostgreSQL
└── database: rpg_book_reaper
    └── schema: public
        ├── table: games
        └── table: llm_models
```

---

## 4. Crear una base de datos

Desde `psql`:

```sql id="svh5wa"
CREATE DATABASE rpg_book_reaper;
```

Conectarse a la base:

```bash id="h8i14z"
psql -U postgres -d rpg_book_reaper
```

`psql` es el cliente de terminal oficial de PostgreSQL; permite ejecutar consultas interactivamente, pasar consultas desde archivos y usar metacomandos útiles para automatizar tareas.

Comandos útiles en `psql`:

```sql id="45xc5w"
\l          -- listar bases de datos
\c nombre   -- conectarse a una base
\dt         -- listar tablas
\d tabla    -- describir una tabla
\du         -- listar roles
\q          -- salir
```

---

## 5. Crear tablas

Ejemplo básico:

```sql id="jt74xe"
CREATE TABLE games (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    complexity TEXT NOT NULL,
    company TEXT NOT NULL,
    ogl_url TEXT
);
```

Ejemplo para modelos LLM:

```sql id="8rjf3r"
CREATE TABLE llm_models (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    size TEXT NOT NULL,
    url TEXT NOT NULL
);
```

---

## 6. Tipos de datos comunes

Tipos frecuentes en PostgreSQL:

```text id="0wh5xb"
INTEGER       # entero normal
BIGINT        # entero grande
SERIAL        # entero autoincremental tradicional
BIGSERIAL     # entero grande autoincremental
UUID          # identificador universal
TEXT          # texto variable
VARCHAR(n)    # texto con límite de longitud
BOOLEAN       # verdadero/falso
DATE          # fecha
TIME          # hora
TIMESTAMP     # fecha y hora sin zona horaria
TIMESTAMPTZ   # fecha y hora con zona horaria
NUMERIC       # número decimal exacto
JSONB         # documento JSON binario indexable
```

Recomendaciones:

```text id="k4nc9s"
Usa TEXT salvo que realmente necesites limitar longitud.
Usa NUMERIC para dinero o valores donde no quieres errores de coma flotante.
Usa TIMESTAMPTZ para fechas de eventos reales.
Usa UUID cuando necesites IDs difíciles de adivinar o generados fuera de la base.
Usa JSONB para datos semiestructurados, pero no para reemplazar todo el modelo relacional.
```

---

## 7. Constraints: reglas de integridad

Las constraints impiden guardar datos inválidos.

### PRIMARY KEY

Identifica de manera única cada fila.

```sql id="pdm6pk"
id SERIAL PRIMARY KEY
```

### NOT NULL

Obliga a que un campo tenga valor.

```sql id="pnwn4b"
name TEXT NOT NULL
```

### UNIQUE

Evita valores duplicados.

```sql id="xl1l5d"
email TEXT UNIQUE
```

### CHECK

Valida una condición.

```sql id="3rgxth"
complexity TEXT NOT NULL CHECK (complexity IN ('Low', 'Medium', 'High'))
```

### FOREIGN KEY

Relaciona una tabla con otra.

```sql id="b85ji4"
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    game_id INTEGER NOT NULL REFERENCES games(id),
    title TEXT NOT NULL
);
```

Regla práctica:

```text id="d1x8nf"
No confíes solo en la API para validar datos.
La base de datos también debe proteger la integridad.
```

---

## 8. Insertar datos

```sql id="qvlsvt"
INSERT INTO games (
    name,
    description,
    complexity,
    company,
    ogl_url
)
VALUES (
    'Daggerheart',
    'A fantasy role-playing game set in a world of magic and monsters.',
    'Medium',
    'Daggerheart Games',
    'https://www.daggerheartgames.com/'
);
```

Insertar varios registros:

```sql id="nq0kqu"
INSERT INTO llm_models (name, description, size, url)
VALUES
    ('Llama', 'Open-weight language model family.', '7B', 'https://huggingface.co/meta-llama/Llama-2-7b-hf'),
    ('Mistral', 'Open-weight language model family.', '7B', 'https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.1');
```

---

## 9. Consultar datos con `SELECT`

Consultar todo:

```sql id="0m5rz8"
SELECT *
FROM games;
```

Consultar columnas específicas:

```sql id="g7je07"
SELECT id, name, complexity
FROM games;
```

Filtrar:

```sql id="yoh2t2"
SELECT id, name, company
FROM games
WHERE complexity = 'Medium';
```

Búsqueda parcial:

```sql id="jjmf81"
SELECT id, name
FROM games
WHERE name ILIKE '%dagger%';
```

Ordenar:

```sql id="5kk006"
SELECT id, name, complexity
FROM games
ORDER BY name ASC;
```

Limitar resultados:

```sql id="eq9z2j"
SELECT id, name
FROM games
LIMIT 10 OFFSET 0;
```

---

## 10. Actualizar datos

```sql id="m3hkra"
UPDATE games
SET complexity = 'High'
WHERE id = 16;
```

Regla importante:

```text id="1n0rox"
Nunca ejecutes UPDATE sin WHERE salvo que quieras modificar toda la tabla.
```

Ejemplo peligroso:

```sql id="zpujg7"
UPDATE games
SET complexity = 'High';
```

Eso modifica todas las filas.

---

## 11. Eliminar datos

```sql id="ubyh1d"
DELETE FROM games
WHERE id = 16;
```

Regla importante:

```text id="jktrv2"
Nunca ejecutes DELETE sin WHERE salvo que quieras borrar todos los registros.
```

Ejemplo peligroso:

```sql id="1wfamc"
DELETE FROM games;
```

---

## 12. Transacciones

Una transacción agrupa operaciones que deben completarse juntas.

```sql id="4g2s1t"
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

Si algo falla:

```sql id="ixovbt"
ROLLBACK;
```

Uso típico:

```text id="codg6b"
BEGIN     # inicia la transacción
COMMIT    # confirma los cambios
ROLLBACK  # revierte los cambios
```

Buenas prácticas:

```text id="l7v7jb"
Usa transacciones para operaciones relacionadas.
Evita transacciones largas innecesarias.
No dejes sesiones abiertas con transacciones pendientes.
```

---

# Parte II: nivel intermedio

## 13. Diseño de bases de datos

Antes de crear tablas, identifica:

```text id="n4pruf"
1. Entidades principales.
2. Atributos de cada entidad.
3. Relaciones entre entidades.
4. Reglas de negocio.
5. Consultas que necesitarás hacer con frecuencia.
```

Ejemplo para una app de RPG:

```text id="4s7hcu"
Entidad: Game
- id
- name
- description
- complexity
- company
- ogl_url

Entidad: Book
- id
- game_id
- title
- edition
- publication_year
- url

Relación:
Un game puede tener muchos books.
Un book pertenece a un game.
```

Modelo SQL:

```sql id="k2l908"
CREATE TABLE games (
    id BIGSERIAL PRIMARY KEY,
    name TEXT NOT NULL UNIQUE,
    description TEXT,
    complexity TEXT NOT NULL CHECK (complexity IN ('Low', 'Medium', 'High')),
    company TEXT NOT NULL,
    ogl_url TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE books (
    id BIGSERIAL PRIMARY KEY,
    game_id BIGINT NOT NULL REFERENCES games(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    edition TEXT,
    publication_year INTEGER CHECK (publication_year >= 1970),
    url TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## 14. Normalización

La normalización reduce duplicación y mejora integridad.

### Mala práctica

```text id="h6pec2"
books
- id
- title
- game_name
- game_company
- game_complexity
```

Aquí `game_company` y `game_complexity` se repetirían en muchos libros.

### Mejor práctica

```text id="en41gv"
games
- id
- name
- company
- complexity

books
- id
- game_id
- title
```

Consulta con relación:

```sql id="lomlzg"
SELECT
    books.title,
    games.name AS game_name,
    games.company
FROM books
JOIN games ON games.id = books.game_id;
```

Regla práctica:

```text id="w3v6mj"
Normaliza primero.
Desnormaliza solo cuando tengas una razón clara de rendimiento o reporting.
```

---

## 15. Relaciones comunes

### Uno a muchos

Un juego tiene muchos libros.

```sql id="le1n0i"
CREATE TABLE books (
    id BIGSERIAL PRIMARY KEY,
    game_id BIGINT NOT NULL REFERENCES games(id),
    title TEXT NOT NULL
);
```

### Muchos a muchos

Un libro puede tener muchos autores y un autor puede escribir muchos libros.

```sql id="52ywsl"
CREATE TABLE authors (
    id BIGSERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

CREATE TABLE book_authors (
    book_id BIGINT NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    author_id BIGINT NOT NULL REFERENCES authors(id) ON DELETE CASCADE,
    PRIMARY KEY (book_id, author_id)
);
```

### Uno a uno

Una tabla de configuración por usuario.

```sql id="3hxpg5"
CREATE TABLE user_profiles (
    user_id BIGINT PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    bio TEXT,
    avatar_url TEXT
);
```

---

## 16. Índices

Un índice acelera consultas, pero también agrega costo en escrituras y ocupa espacio. PostgreSQL indica que los índices permiten encontrar filas más rápido, pero deben usarse con criterio porque agregan sobrecarga al sistema.

Ejemplo:

```sql id="sw5q5o"
CREATE INDEX idx_games_name
ON games (name);
```

Índice para búsquedas por complejidad:

```sql id="e4nrcx"
CREATE INDEX idx_games_complexity
ON games (complexity);
```

Índice compuesto:

```sql id="phmvzs"
CREATE INDEX idx_games_company_complexity
ON games (company, complexity);
```

PostgreSQL soporta varios tipos de índices, incluyendo B-tree, Hash, GiST, SP-GiST, GIN, BRIN y la extensión bloom. Por defecto, `CREATE INDEX` crea índices B-tree, adecuados para muchos casos comunes.

Uso típico:

```text id="jyyug5"
B-tree  # igualdad, rangos, ORDER BY
GIN     # JSONB, arrays, full-text search
GiST    # geometría, rangos, búsquedas especiales
BRIN    # tablas muy grandes ordenadas físicamente por algún criterio
Hash    # igualdad, menos común que B-tree
```

---

## 17. No indexes todo

Evita crear índices sin necesidad.

Cada índice:

```text id="o610na"
Acelera algunas lecturas.
Ralentiza INSERT, UPDATE y DELETE.
Consume espacio en disco.
Debe mantenerse actualizado.
```

Antes de crear un índice, pregunta:

```text id="cjflbs"
¿Esta columna se usa en WHERE?
¿Esta columna se usa en JOIN?
¿Esta columna se usa en ORDER BY?
¿La tabla tiene suficientes filas para justificar el índice?
¿La consulta es frecuente o crítica?
```

---

## 18. Analizar consultas con EXPLAIN

Usa `EXPLAIN` para ver el plan de ejecución de una consulta. PostgreSQL documenta que `EXPLAIN` muestra cómo el planner ejecutará una consulta, incluyendo si usará sequential scan, index scan u otros métodos.

Ejemplo:

```sql id="j3yye6"
EXPLAIN
SELECT *
FROM games
WHERE name = 'Daggerheart';
```

Para ejecutar la consulta y ver tiempos reales:

```sql id="xgic39"
EXPLAIN ANALYZE
SELECT *
FROM games
WHERE name = 'Daggerheart';
```

Precaución:

```text id="708uvc"
EXPLAIN solo estima.
EXPLAIN ANALYZE ejecuta la consulta realmente.
No uses EXPLAIN ANALYZE con DELETE, UPDATE o INSERT destructivos sin una transacción controlada.
```

Ejemplo seguro:

```sql id="qoqj7j"
BEGIN;

EXPLAIN ANALYZE
DELETE FROM games
WHERE id = 999;

ROLLBACK;
```

---

## 19. Vistas

Una vista guarda una consulta reutilizable.

```sql id="tprl3w"
CREATE VIEW medium_complexity_games AS
SELECT id, name, company
FROM games
WHERE complexity = 'Medium';
```

Consultar:

```sql id="s67wnq"
SELECT *
FROM medium_complexity_games;
```

Las vistas son útiles para:

```text id="4p8k35"
Simplificar consultas frecuentes.
Exponer datos limitados.
Encapsular lógica SQL.
Crear interfaces estables para reportes.
```

---

## 20. Materialized views

Una materialized view guarda físicamente el resultado de una consulta.

```sql id="9dn9xe"
CREATE MATERIALIZED VIEW game_book_counts AS
SELECT
    games.id,
    games.name,
    COUNT(books.id) AS book_count
FROM games
LEFT JOIN books ON books.game_id = games.id
GROUP BY games.id, games.name;
```

Actualizar:

```sql id="i9ty6l"
REFRESH MATERIALIZED VIEW game_book_counts;
```

Uso recomendado:

```text id="6hljmh"
Reportes costosos.
Consultas agregadas pesadas.
Datos que no necesitan estar en tiempo real.
```

---

## 21. Migraciones

Una migración es un cambio versionado en el esquema de la base de datos.

Ejemplos:

```text id="pgc76a"
Crear tabla.
Agregar columna.
Cambiar tipo de dato.
Crear índice.
Agregar constraint.
Eliminar columna.
```

Buenas prácticas:

```text id="k8fjww"
No modifiques la base de datos manualmente en producción.
Versiona los cambios.
Usa herramientas como Alembic, Flyway, Liquibase o Django migrations.
Revisa el SQL generado antes de aplicarlo.
Ten un plan de rollback.
Prueba migraciones con una copia de datos reales o representativos.
```

Ejemplo conceptual:

```text id="7qntlj"
001_create_games_table.sql
002_create_books_table.sql
003_add_ogl_url_to_games.sql
004_create_index_on_games_name.sql
```

---

## 22. Seguridad: roles y privilegios

No uses el superusuario de PostgreSQL desde tu aplicación.

Crea un rol específico:

```sql id="mdtthh"
CREATE ROLE app_user
WITH LOGIN PASSWORD 'change_this_password';
```

Crea base de datos con dueño:

```sql id="2fqqlu"
CREATE DATABASE rpg_book_reaper
OWNER app_user;
```

Otorga permisos limitados:

```sql id="4q3jyy"
GRANT CONNECT ON DATABASE rpg_book_reaper TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_user;
```

PostgreSQL maneja permisos mediante privileges; la documentación indica que no se otorgan permisos por defecto a `PUBLIC` sobre tablas, columnas, secuencias, esquemas, tablespaces ni varios otros objetos cuando estos se crean.

Reglas:

```text id="2dxq91"
Un usuario para administrar.
Otro usuario para la aplicación.
Otro usuario de solo lectura para BI/reportes.
Nunca uses superuser en la aplicación.
No guardes contraseñas en el repositorio.
```

---

## 23. Backups básicos

PostgreSQL tiene varias estrategias de backup. A nivel general, existen backups lógicos, backups físicos y recuperación continua con WAL/PITR. La documentación oficial describe tres enfoques generales: SQL dump, backup a nivel de sistema de archivos y backup online.

### Backup lógico con `pg_dump`

```bash id="slbhb7"
pg_dump -U app_user -d rpg_book_reaper -Fc -f rpg_book_reaper.dump
```

La documentación de `pg_dump` indica que el formato custom `-Fc`, usado junto con `pg_restore`, entrega un mecanismo flexible para archivar y restaurar bases de datos.

### Restaurar con `pg_restore`

```bash id="7fvyr6"
createdb -U postgres rpg_book_reaper_restore

pg_restore -U postgres -d rpg_book_reaper_restore rpg_book_reaper.dump
```

`pg_restore` restaura una base desde un archivo creado por `pg_dump` en formatos no planos y permite reconstruir la base al estado del momento del respaldo.

### Backup de todo el cluster con `pg_dumpall`

```bash id="sbsq9n"
pg_dumpall -U postgres > full_cluster_backup.sql
```

`pg_dumpall` genera un script SQL para todas las bases de datos de un cluster y también incluye objetos globales como roles, tablespaces y ciertos grants que `pg_dump` no guarda.

Regla crítica:

```text id="k6rxz2"
Un backup que nunca has restaurado es solo una esperanza, no una garantía.
```

---

## 24. Mantención: VACUUM y ANALYZE

PostgreSQL usa MVCC, por lo que las actualizaciones y eliminaciones pueden dejar filas muertas que deben limpiarse.

`VACUUM` ayuda a limpiar filas muertas.

```sql id="dvxvtg"
VACUUM;
```

`ANALYZE` actualiza estadísticas para que el planner elija mejores planes.

```sql id="jjusvg"
ANALYZE;
```

Combinado:

```sql id="okdtsc"
VACUUM ANALYZE;
```

La documentación oficial recomienda vacuum regular para remover filas muertas, y PostgreSQL incluye autovacuum para automatizar esta mantención.

El objetivo normal del vacuum rutinario no es reducir las tablas al tamaño mínimo, sino mantener un uso estable del espacio y evitar necesitar `VACUUM FULL`; autovacuum trabaja bajo esa idea y no ejecuta `VACUUM FULL`.

---

## 25. Evita abusar de VACUUM FULL

`VACUUM FULL` puede recuperar espacio físico, pero es una operación más invasiva.

Regla práctica:

```text id="u8fss7"
Usa VACUUM normal para mantenimiento rutinario.
Usa VACUUM FULL solo cuando sepas por qué lo necesitas.
Planifica VACUUM FULL en ventanas de mantenimiento.
```

---

## 26. Conexiones y pooling

No abras una conexión nueva por cada request si tu aplicación tiene tráfico.

Usa un pool de conexiones:

```text id="h8xjjc"
FastAPI + SQLAlchemy
FastAPI + asyncpg
PgBouncer
Pool interno del framework
```

Problemas comunes por no usar pooling:

```text id="da8py8"
Demasiadas conexiones abiertas.
Consumo alto de memoria.
Errores por max_connections.
Latencia innecesaria.
```

Regla práctica:

```text id="rky2f6"
La aplicación debe reutilizar conexiones.
La base de datos no debe recibir conexiones infinitas.
```

---

## 27. SQL injection

Nunca construyas SQL concatenando strings con datos del usuario.

Mala práctica:

```python id="e1z52r"
query = f"SELECT * FROM users WHERE email = '{email}'"
```

Mejor:

```python id="dwvr7q"
query = "SELECT * FROM users WHERE email = %s"
cursor.execute(query, (email,))
```

Con SQLAlchemy:

```python id="6tcxp4"
stmt = select(User).where(User.email == email)
```

Regla:

```text id="a31f4z"
Los valores del usuario deben ir como parámetros, no como SQL interpolado.
```

---

# Parte III: nivel avanzado

## 28. Particionamiento

El particionamiento divide una tabla grande en partes más pequeñas.

Casos típicos:

```text id="78z6j8"
Eventos por fecha.
Logs por mes.
Transacciones por año.
Datos multi-tenant por cliente.
```

Ejemplo por rango de fecha:

```sql id="9u4u8z"
CREATE TABLE events (
    id BIGSERIAL,
    event_date DATE NOT NULL,
    payload JSONB NOT NULL,
    PRIMARY KEY (id, event_date)
)
PARTITION BY RANGE (event_date);
```

Crear particiones:

```sql id="hy793c"
CREATE TABLE events_2026_01
PARTITION OF events
FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');

CREATE TABLE events_2026_02
PARTITION OF events
FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');
```

En PostgreSQL, un índice o constraint único declarado sobre una tabla particionada es “virtual”; los datos reales están en los índices hijos de cada partición.

Buenas prácticas:

```text id="gcatqa"
Particiona solo cuando la tabla realmente lo necesita.
Define una estrategia clara de creación y eliminación de particiones.
Asegúrate de que las consultas filtren por la clave de partición.
No uses particionamiento para tablas pequeñas.
```

---

## 29. Write-Ahead Log, WAL y recuperación

WAL es central para la confiabilidad de PostgreSQL. La idea básica es que los cambios en archivos de datos se escriben después de que los registros WAL correspondientes hayan sido guardados en almacenamiento persistente.

PostgreSQL mantiene WAL en el subdirectorio `pg_wal/`; este registro contiene cambios hechos a los archivos de datos y permite recuperar consistencia tras una caída reproduciendo esos registros.

Conceptos relacionados:

```text id="xgguz1"
WAL              # registro de cambios
Checkpoint       # punto de sincronización
Archive mode     # archivado de WAL
PITR             # Point-in-Time Recovery
Replication slot # mecanismo para retener WAL necesario por réplicas
```

Casos de uso:

```text id="c3u63m"
Recuperación tras crash.
Replicación física.
Backup continuo.
Recuperación a un punto específico en el tiempo.
```

---

## 30. Point-in-Time Recovery, PITR

PITR permite restaurar una base a un momento específico.

Uso típico:

```text id="0ckpjd"
1. Tienes un backup base.
2. Tienes archivos WAL archivados.
3. Restauras el backup base.
4. Reproduces WAL hasta un timestamp específico.
```

Ejemplo de caso real:

```text id="57soze"
A las 10:03 alguien ejecuta DELETE FROM users.
Quieres restaurar la base al estado de las 10:02:59.
```

Buenas prácticas:

```text id="0vs1um"
Activa archivado WAL si necesitas recuperación fina.
Prueba restauraciones.
Documenta RPO y RTO.
Monitorea que el archivado WAL no se rompa.
```

---

## 31. Replicación

PostgreSQL soporta varias estrategias de replicación. La documentación describe, entre otras, soluciones basadas en shipping de WAL, donde servidores standby se mantienen al día leyendo registros WAL y pueden promoverse si falla el servidor principal.

Tipos frecuentes:

```text id="jtjjvt"
Replicación física streaming    # réplica binaria del cluster
Replicación lógica              # replica tablas/cambios lógicos
Read replica                    # réplica para lecturas
Hot standby                     # réplica que acepta consultas de solo lectura
```

Usos:

```text id="t6cz88"
Alta disponibilidad.
Escalamiento de lecturas.
Migraciones con menor downtime.
Copias para reporting.
```

Advertencias:

```text id="oljham"
Una réplica no reemplaza un backup.
Una réplica puede replicar errores lógicos, como DELETE accidental.
Monitorea replication lag.
Define procedimientos de failover.
```

---

## 32. Índices avanzados

### Índices parciales

```sql id="x4qxl1"
CREATE INDEX idx_active_users_email
ON users (email)
WHERE deleted_at IS NULL;
```

Útil cuando consultas mucho un subconjunto.

### Índices por expresión

```sql id="sn9ymd"
CREATE INDEX idx_users_lower_email
ON users (lower(email));
```

Consulta:

```sql id="mh8g3e"
SELECT *
FROM users
WHERE lower(email) = lower('USER@example.com');
```

PostgreSQL permite índices sobre expresiones, por ejemplo `upper(col)`, para acelerar búsquedas basadas en transformaciones de columnas.

### Índices GIN para JSONB

```sql id="zqyn1t"
CREATE INDEX idx_events_payload_gin
ON events
USING GIN (payload);
```

Consulta:

```sql id="ow5bp9"
SELECT *
FROM events
WHERE payload @> '{"type": "game_created"}';
```

### BRIN para tablas enormes

```sql id="eychbo"
CREATE INDEX idx_events_event_date_brin
ON events
USING BRIN (event_date);
```

Útil si los datos están físicamente correlacionados con la columna, por ejemplo logs insertados por fecha.

---

## 33. JSONB

JSONB sirve para datos semiestructurados.

Ejemplo:

```sql id="13livt"
CREATE TABLE game_metadata (
    id BIGSERIAL PRIMARY KEY,
    game_id BIGINT NOT NULL REFERENCES games(id),
    metadata JSONB NOT NULL
);
```

Insertar:

```sql id="wzi9hc"
INSERT INTO game_metadata (game_id, metadata)
VALUES (
    1,
    '{"genres": ["fantasy", "adventure"], "dice": "d20"}'
);
```

Consultar:

```sql id="016ckw"
SELECT *
FROM game_metadata
WHERE metadata @> '{"dice": "d20"}';
```

Buenas prácticas:

```text id="y6vzlu"
Usa columnas normales para campos importantes y consultados frecuentemente.
Usa JSONB para atributos variables o poco estructurados.
No conviertas toda la base en documentos JSON si necesitas integridad relacional.
Indexa JSONB si haces consultas frecuentes sobre su contenido.
```

---

## 34. Full-text search

PostgreSQL puede usarse para búsqueda de texto.

Ejemplo:

```sql id="ksdzyx"
SELECT id, name, description
FROM games
WHERE to_tsvector('english', description) @@ plainto_tsquery('english', 'magic monsters');
```

Índice recomendado:

```sql id="1a6b0r"
CREATE INDEX idx_games_description_fts
ON games
USING GIN (to_tsvector('english', description));
```

Uso recomendado:

```text id="rpn41r"
Búsquedas internas simples o medianas.
Búsqueda textual integrada con datos relacionales.
Filtros combinados con SQL.
```

Para búsqueda semántica avanzada o ranking muy especializado, evalúa motores externos o extensiones específicas.

---

## 35. Window functions

Las window functions calculan valores sobre grupos de filas sin colapsarlas.

Ejemplo: ranking de juegos por compañía.

```sql id="a765vq"
SELECT
    id,
    name,
    company,
    complexity,
    ROW_NUMBER() OVER (
        PARTITION BY company
        ORDER BY name
    ) AS company_rank
FROM games;
```

Ejemplo: conteo por compañía manteniendo filas individuales.

```sql id="5p4jdj"
SELECT
    id,
    name,
    company,
    COUNT(*) OVER (
        PARTITION BY company
    ) AS games_by_company
FROM games;
```

---

## 36. CTEs

Una CTE permite dividir consultas complejas.

```sql id="b2tdbf"
WITH medium_games AS (
    SELECT *
    FROM games
    WHERE complexity = 'Medium'
)
SELECT company, COUNT(*) AS total
FROM medium_games
GROUP BY company;
```

Uso recomendado:

```text id="105h8c"
Mejorar legibilidad.
Separar pasos lógicos.
Construir consultas analíticas.
```

---

## 37. Locks y concurrencia

Problemas típicos:

```text id="cwmmth"
Dos usuarios actualizan el mismo dato.
Una migración bloquea una tabla usada por producción.
Una transacción larga impide limpieza por VACUUM.
Un reporte pesado compite con tráfico de aplicación.
```

Buenas prácticas:

```text id="pvu63w"
Mantén transacciones cortas.
Actualiza filas en orden consistente.
Evita migraciones pesadas en horario crítico.
Usa índices para que UPDATE y DELETE encuentren filas rápido.
Monitorea sesiones bloqueadas.
```

Consulta útil:

```sql id="ooeoex"
SELECT
    pid,
    state,
    wait_event_type,
    wait_event,
    query
FROM pg_stat_activity
WHERE state <> 'idle';
```

---

## 38. Tuning de configuración

No existe una configuración universal. Depende de:

```text id="72qzsi"
RAM disponible.
CPU.
Tipo de disco.
Volumen de escritura.
Volumen de lectura.
Tamaño de datos.
Número de conexiones.
Tipo de workload: OLTP, OLAP, mixto.
```

Parámetros relevantes:

```text id="nnklpm"
shared_buffers
work_mem
maintenance_work_mem
effective_cache_size
max_connections
checkpoint_timeout
max_wal_size
autovacuum_work_mem
random_page_cost
effective_io_concurrency
```

La documentación de PostgreSQL señala que valores mayores de memoria de mantenimiento pueden mejorar rendimiento en vacuum y restauración de dumps, pero también advierte que autovacuum puede multiplicar ese uso según sus workers.

Regla práctica:

```text id="6v2ylt"
No copies configuraciones de internet sin medir.
Cambia un parámetro a la vez.
Documenta cada cambio.
Mide antes y después.
```

---

## 39. Observabilidad

Debes monitorear como mínimo:

```text id="jvpjy8"
Uso de CPU.
Uso de memoria.
Uso de disco.
Crecimiento de tablas.
Crecimiento de índices.
Número de conexiones.
Consultas lentas.
Bloqueos.
Replication lag.
Errores de backup.
Autovacuum.
```

Vistas útiles:

```sql id="e7mquu"
SELECT *
FROM pg_stat_activity;
```

```sql id="ktj6lj"
SELECT *
FROM pg_stat_user_tables;
```

```sql id="eftz7u"
SELECT *
FROM pg_stat_user_indexes;
```

```sql id="cdxw3m"
SELECT *
FROM pg_stat_database;
```

PostgreSQL incluye vistas de progreso como `pg_stat_progress_analyze`, `pg_stat_progress_create_index` y `pg_stat_progress_vacuum`, útiles para observar operaciones en curso.

---

## 40. Logs y consultas lentas

Configura logs para detectar consultas lentas.

Parámetros relevantes:

```text id="ajpzae"
log_min_duration_statement
log_lock_waits
deadlock_timeout
log_checkpoints
log_connections
log_disconnections
```

Ejemplo conceptual:

```conf id="0b516h"
log_min_duration_statement = 500
log_lock_waits = on
```

Esto permite registrar consultas que tarden más de 500 ms.

También puedes usar `auto_explain`, que registra automáticamente planes de ejecución de consultas lentas sin ejecutar `EXPLAIN` manualmente.

---

## 41. Extensiones útiles

PostgreSQL permite instalar extensiones.

Ejemplos:

```sql id="q65d7e"
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

Extensiones comunes:

```text id="kn8n10"
pg_stat_statements  # estadísticas de consultas
pgcrypto            # funciones criptográficas
uuid-ossp           # generación de UUIDs
citext              # texto case-insensitive
postgis             # datos geoespaciales
pg_trgm             # búsqueda por similitud textual
```

La documentación oficial agrupa muchas extensiones en módulos adicionales distribuidos con PostgreSQL.

---

## 42. Row-Level Security

Row-Level Security, o RLS, permite restringir filas según políticas.

Ejemplo:

```sql id="jef95c"
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;
```

Política:

```sql id="omjxca"
CREATE POLICY documents_owner_policy
ON documents
FOR SELECT
USING (owner_id = current_setting('app.current_user_id')::BIGINT);
```

Uso típico:

```text id="1tq0qy"
Aplicaciones multi-tenant.
Datos por organización.
Datos por usuario.
Sistemas con permisos finos.
```

Advertencia:

```text id="ubg9ig"
RLS es potente, pero aumenta complejidad.
Documenta muy bien las políticas.
Prueba casos permitidos y prohibidos.
```

---

## 43. Multi-tenancy

Modelos comunes:

### Una base por cliente

```text id="zdur5j"
Cliente A -> database_a
Cliente B -> database_b
```

Ventajas:

```text id="u91ni2"
Aislamiento fuerte.
Backups por cliente.
Restauración más simple por cliente.
```

Desventajas:

```text id="z5niyn"
Operación más compleja.
Migraciones repetidas.
Más conexiones y recursos.
```

### Un schema por cliente

```text id="p6rr3x"
tenant_a.games
tenant_b.games
```

Ventajas:

```text id="4hbht0"
Aislamiento medio.
Una sola base.
Separación lógica clara.
```

Desventajas:

```text id="yqv4iu"
Migraciones más complejas.
Muchos schemas pueden complicar operación.
```

### Una tabla compartida con `tenant_id`

```sql id="vgfdkg"
CREATE TABLE games (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL,
    name TEXT NOT NULL
);
```

Ventajas:

```text id="j8pecx"
Modelo simple.
Escala bien para muchos tenants pequeños.
Migraciones más simples.
```

Desventajas:

```text id="2tqaqd"
Riesgo de fuga de datos si olvidas filtrar por tenant_id.
Requiere disciplina, constraints y posiblemente RLS.
```

---

## 44. Estrategias de borrado: físico vs lógico

### Borrado físico

```sql id="8esxae"
DELETE FROM games
WHERE id = 16;
```

Ventajas:

```text id="pbl8u5"
Simple.
Libera datos lógicamente.
Menos complejidad.
```

Desventajas:

```text id="no0lnc"
Difícil recuperar datos.
Puede afectar auditoría.
```

### Borrado lógico

```sql id="rc7bpg"
ALTER TABLE games
ADD COLUMN deleted_at TIMESTAMPTZ;
```

Borrar:

```sql id="0lqvi4"
UPDATE games
SET deleted_at = now()
WHERE id = 16;
```

Consultar activos:

```sql id="6gfkw9"
SELECT *
FROM games
WHERE deleted_at IS NULL;
```

Ventajas:

```text id="65vefq"
Permite recuperación.
Útil para auditoría.
Evita borrar datos por accidente.
```

Desventajas:

```text id="e8h9rs"
Todas las consultas deben filtrar deleted_at.
Las tablas crecen más.
Puede requerir índices parciales.
```

Índice parcial útil:

```sql id="wgt8ug"
CREATE INDEX idx_games_active_name
ON games (name)
WHERE deleted_at IS NULL;
```

---

## 45. Auditoría

Campos básicos recomendados:

```sql id="8wmpqg"
created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
deleted_at TIMESTAMPTZ
```

Opcional:

```sql id="nln6vt"
created_by BIGINT,
updated_by BIGINT,
deleted_by BIGINT
```

Para auditoría fuerte:

```text id="gcx9sr"
Usa tablas de historial.
Registra cambios relevantes.
Registra usuario, timestamp y operación.
No dependas solo de logs de aplicación.
```

Ejemplo simple:

```sql id="irrmxy"
CREATE TABLE audit_log (
    id BIGSERIAL PRIMARY KEY,
    table_name TEXT NOT NULL,
    row_id BIGINT NOT NULL,
    action TEXT NOT NULL CHECK (action IN ('INSERT', 'UPDATE', 'DELETE')),
    changed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    changed_by TEXT,
    old_data JSONB,
    new_data JSONB
);
```

---

# Parte IV: mantención operativa

## 46. Rutina diaria

```text id="drvk3e"
[ ] Verificar que los backups terminaron correctamente.
[ ] Verificar espacio en disco.
[ ] Revisar errores recientes en logs.
[ ] Revisar consultas lentas.
[ ] Revisar conexiones activas.
[ ] Revisar replication lag si hay réplicas.
[ ] Revisar alertas de monitoreo.
```

---

## 47. Rutina semanal

```text id="vfh7ed"
[ ] Probar restauración de backup en ambiente aislado.
[ ] Revisar crecimiento de tablas e índices.
[ ] Revisar tablas con muchas filas muertas.
[ ] Revisar índices no usados.
[ ] Revisar cambios de permisos.
[ ] Revisar jobs programados.
[ ] Revisar tiempos de queries críticas.
```

---

## 48. Rutina mensual

```text id="a1nso7"
[ ] Revisar estrategia de backup y retención.
[ ] Revisar plan de recuperación ante desastre.
[ ] Revisar usuarios y roles.
[ ] Revisar extensiones instaladas.
[ ] Revisar configuración de autovacuum.
[ ] Revisar necesidad de particionamiento.
[ ] Revisar upgrades menores pendientes.
```

La política de versionado de PostgreSQL indica que se publica una versión mayor aproximadamente una vez al año y que cada versión mayor recibe correcciones de bugs y seguridad en releases menores al menos una vez cada tres meses.

---

## 49. Backups: estrategia recomendada

Para producción, combina:

```text id="tt58wb"
1. Backups lógicos para portabilidad.
2. Backups físicos para restauración rápida.
3. WAL archiving para PITR.
4. Pruebas regulares de restauración.
```

Define:

```text id="cuwklx"
RPO: cuántos datos puedes perder.
RTO: cuánto tiempo puedes tardar en restaurar.
Retención: cuántos días/semanas/meses guardarás backups.
Cifrado: cómo protegerás los respaldos.
Ubicación: dónde guardarás respaldos fuera del servidor principal.
```

Ejemplo de política:

```text id="g96sa9"
Backup lógico diario.
Backup físico semanal.
WAL archive continuo.
Retención diaria por 14 días.
Retención semanal por 8 semanas.
Retención mensual por 12 meses.
Prueba de restore mensual.
```

---

## 50. Upgrades

Tipos:

```text id="qnsubp"
Minor upgrade: 18.3 -> 18.4
Major upgrade: 17 -> 18
```

Buenas prácticas:

```text id="hlx3u1"
Lee release notes.
Prueba el upgrade en staging.
Haz backup antes.
Verifica extensiones.
Mide queries críticas antes y después.
Ten plan de rollback.
Agenda ventana de mantenimiento si aplica.
```

PostgreSQL 18 incorporó mejoras como un subsistema de I/O asíncrono, conservación de estadísticas del optimizador con `pg_upgrade`, skip scan para ciertos índices B-tree multicolumna y la función `uuidv7()`. Estas características dependen de versión, por lo que conviene leer las release notes antes de adoptar una rama nueva.

---

# Parte V: ejemplo aplicado a una API FastAPI

## 51. Modelo recomendado para `games` y `llm_models`

```sql id="xgoq46"
CREATE TABLE games (
    id BIGSERIAL PRIMARY KEY,
    name TEXT NOT NULL UNIQUE,
    description TEXT,
    complexity TEXT NOT NULL CHECK (complexity IN ('Low', 'Medium', 'High')),
    company TEXT NOT NULL,
    ogl_url TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at TIMESTAMPTZ
);

CREATE TABLE llm_models (
    id BIGSERIAL PRIMARY KEY,
    name TEXT NOT NULL UNIQUE,
    description TEXT,
    size TEXT NOT NULL,
    url TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at TIMESTAMPTZ
);
```

Índices:

```sql id="fpyf1e"
CREATE INDEX idx_games_complexity
ON games (complexity)
WHERE deleted_at IS NULL;

CREATE INDEX idx_games_company
ON games (company)
WHERE deleted_at IS NULL;

CREATE INDEX idx_llm_models_size
ON llm_models (size)
WHERE deleted_at IS NULL;
```

---

## 52. Consultas típicas para tu API

### Listar juegos activos

```sql id="0jeidl"
SELECT
    id,
    name,
    description,
    complexity,
    company,
    ogl_url
FROM games
WHERE deleted_at IS NULL
ORDER BY name;
```

### Buscar juegos por nombre

```sql id="55l4sq"
SELECT
    id,
    name,
    description,
    complexity,
    company,
    ogl_url
FROM games
WHERE deleted_at IS NULL
  AND name ILIKE '%' || $1 || '%'
ORDER BY name;
```

### Buscar por ID

```sql id="gssha6"
SELECT
    id,
    name,
    description,
    complexity,
    company,
    ogl_url
FROM games
WHERE id = $1
  AND deleted_at IS NULL;
```

### Crear modelo LLM

```sql id="bbht0x"
INSERT INTO llm_models (
    name,
    description,
    size,
    url
)
VALUES (
    $1,
    $2,
    $3,
    $4
)
RETURNING
    id,
    name,
    description,
    size,
    url,
    created_at,
    updated_at;
```

### Actualizar modelo LLM

```sql id="ttmr7c"
UPDATE llm_models
SET
    name = COALESCE($2, name),
    description = COALESCE($3, description),
    size = COALESCE($4, size),
    url = COALESCE($5, url),
    updated_at = now()
WHERE id = $1
  AND deleted_at IS NULL
RETURNING
    id,
    name,
    description,
    size,
    url,
    created_at,
    updated_at;
```

### Borrado lógico

```sql id="1rrsww"
UPDATE llm_models
SET deleted_at = now()
WHERE id = $1
  AND deleted_at IS NULL;
```

---

# Parte VI: errores comunes

## 53. Errores de principiante

```text id="90enlv"
Usar SELECT * en todas partes.
No definir primary keys.
No definir foreign keys.
Guardar todo como TEXT.
No usar NOT NULL.
No usar UNIQUE donde corresponde.
No hacer backups.
No probar restauraciones.
Usar el usuario postgres desde la aplicación.
Concatenar SQL con datos del usuario.
```

---

## 54. Errores intermedios

```text id="7b2xaz"
Crear índices sin medir.
No revisar EXPLAIN.
No versionar migraciones.
Hacer migraciones pesadas en horario productivo.
No monitorear autovacuum.
No controlar número de conexiones.
Usar JSONB para todo.
No definir estrategia de borrado.
No separar usuarios de lectura y escritura.
```

---

## 55. Errores avanzados

```text id="hl475y"
Creer que replicación reemplaza backups.
No monitorear replication lag.
No probar failover.
No documentar RPO/RTO.
Particionar tablas pequeñas.
No archivar WAL correctamente.
No probar PITR.
Ignorar locks.
Cambiar parámetros de tuning sin medir.
No leer release notes antes de upgrades mayores.
```

---

# Parte VII: checklist general

## 56. Checklist de diseño

```text id="apjgjs"
[ ] ¿Cada tabla tiene primary key?
[ ] ¿Las relaciones tienen foreign keys?
[ ] ¿Los campos obligatorios tienen NOT NULL?
[ ] ¿Los valores únicos tienen UNIQUE?
[ ] ¿Las reglas de negocio críticas tienen CHECK?
[ ] ¿Las fechas usan TIMESTAMPTZ cuando corresponde?
[ ] ¿Los nombres de tablas están en plural?
[ ] ¿Los nombres de columnas son consistentes?
[ ] ¿El esquema evita duplicación innecesaria?
[ ] ¿Hay una estrategia clara para borrado físico o lógico?
```

---

## 57. Checklist de uso

```text id="q1b6d5"
[ ] ¿Las consultas usan parámetros?
[ ] ¿Los UPDATE tienen WHERE?
[ ] ¿Los DELETE tienen WHERE?
[ ] ¿Las operaciones relacionadas usan transacciones?
[ ] ¿La aplicación usa pool de conexiones?
[ ] ¿Las consultas frecuentes tienen índices adecuados?
[ ] ¿Las queries críticas fueron revisadas con EXPLAIN?
[ ] ¿La API no usa usuario superuser?
```

---

## 58. Checklist de mantención

```text id="wnh2h3"
[ ] ¿Hay backups automáticos?
[ ] ¿Los backups se restauran en pruebas?
[ ] ¿Hay monitoreo de disco?
[ ] ¿Hay monitoreo de conexiones?
[ ] ¿Hay monitoreo de queries lentas?
[ ] ¿Autovacuum está funcionando correctamente?
[ ] ¿Se revisan tablas e índices grandes?
[ ] ¿Se revisan usuarios y permisos?
[ ] ¿Se aplican actualizaciones menores?
[ ] ¿Existe plan de recuperación ante desastre?
```

---

## 59. Checklist de producción

```text id="97guwk"
[ ] ¿La base está en disco persistente y confiable?
[ ] ¿Hay backups fuera del servidor principal?
[ ] ¿Hay cifrado en tránsito?
[ ] ¿Hay cifrado o protección adecuada en reposo?
[ ] ¿Las credenciales están en variables de entorno o secret manager?
[ ] ¿El usuario de la app tiene permisos mínimos?
[ ] ¿Hay límites de conexión?
[ ] ¿Hay alertas de disco, CPU, memoria y errores?
[ ] ¿Hay estrategia de upgrade?
[ ] ¿Hay documentación operativa?
```

---

# Parte VIII: reglas finales

## 60. Reglas principales

```text id="f82q9x"
1. Diseña primero, crea tablas después.
2. Usa constraints para proteger la integridad.
3. Usa foreign keys para relaciones reales.
4. No uses superuser desde la aplicación.
5. No concatenes SQL con input del usuario.
6. Usa migraciones versionadas.
7. Crea índices con base en consultas reales.
8. Mide con EXPLAIN antes de optimizar.
9. Configura backups desde el inicio.
10. Prueba restauraciones regularmente.
11. Mantén transacciones cortas.
12. Monitorea queries lentas, locks y crecimiento.
13. No confundas replicación con backup.
14. Lee release notes antes de upgrades.
15. Documenta decisiones de diseño y operación.
```

---

## 61. Ruta de aprendizaje recomendada

```text id="wge7cm"
Nivel 1:
- SELECT, INSERT, UPDATE, DELETE
- WHERE, ORDER BY, LIMIT
- JOIN
- PRIMARY KEY, FOREIGN KEY, NOT NULL, UNIQUE

Nivel 2:
- Normalización
- Índices
- EXPLAIN
- Transacciones
- Migraciones
- Roles y permisos
- Backups con pg_dump y pg_restore

Nivel 3:
- VACUUM y autovacuum
- Particionamiento
- Replicación
- WAL y PITR
- Tuning
- Observabilidad
- Locks
- Alta disponibilidad
- Seguridad avanzada
```

---

## 62. Resumen

Una buena base de datos PostgreSQL no depende solo de escribir SQL correcto. También requiere:

```text id="vuu22n"
Buen diseño.
Validación en la base.
Índices adecuados.
Permisos mínimos.
Backups probados.
Mantenimiento rutinario.
Monitoreo.
Migraciones controladas.
Plan de recuperación.
```

Para un proyecto pequeño, lo más importante es empezar con tablas claras, constraints, backups y migraciones. Para un proyecto mediano, agrega índices medidos, monitoreo, roles separados y pruebas de restauración. Para producción seria, incorpora replicación, PITR, observabilidad, tuning documentado, control de locks, estrategia de upgrades y procedimientos de recuperación ante desastre.

---

## Referencias oficiales consultadas

* Documentación actual de PostgreSQL 18.4.
* Política de versionado de PostgreSQL.
* Documentación de `psql`.
* Documentación de privilegios.
* Documentación de índices.
* Documentación de `EXPLAIN`.
* Documentación de `VACUUM` y autovacuum.
* Documentación de `pg_dump`, `pg_restore` y `pg_dumpall`.
* Documentación de particionamiento.
* Documentación de WAL y recuperación continua.
* Documentación de replicación.
