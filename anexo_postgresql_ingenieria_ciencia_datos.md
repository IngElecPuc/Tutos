# Anexo A: PostgreSQL para ingeniería y ciencia de datos

## A.1. Objetivo del anexo

Este anexo complementa la guía principal de PostgreSQL con prácticas orientadas a ingeniería de datos, analítica, ciencia de datos y sistemas con búsqueda semántica. La guía principal ya cubre fundamentos SQL, diseño relacional, índices, seguridad, backups, particionamiento, observabilidad y mantenimiento operativo; este anexo toma esos conceptos y los aplica a flujos de datos analíticos.

El foco de este anexo es responder preguntas prácticas:

```text
¿Cómo cargo datos de forma repetible?
¿Cuándo conviene ETL y cuándo ELT?
¿Cómo separo datos crudos, datos limpios y datos analíticos?
¿Cómo modelo un data warehouse en PostgreSQL?
¿Cómo conecto PostgreSQL con datalakes y lakehouses?
¿Cómo gobierno datos sensibles?
¿Cómo uso PostgreSQL como base vectorial con pgvector?
¿Cómo diseño pipelines que puedan fallar y reintentarse sin romper datos?
```

PostgreSQL puede cumplir varios roles dentro de una arquitectura de datos:

```text
Base transaccional           # OLTP: datos operativos de una aplicación
Base analítica pequeña/media # reportes, dashboards, análisis internos
Capa de staging              # zona temporal de carga y validación
Data mart                    # subconjunto curado para un equipo o dominio
Metadata store               # control de pipelines, catálogos, auditoría
Vector store                 # embeddings, búsqueda semántica y RAG con pgvector
```

No debe asumirse que PostgreSQL reemplaza siempre a un data warehouse distribuido, un datalake o un motor MPP. Para volúmenes grandes, consultas masivas sobre archivos columnares o procesamiento distribuido, conviene evaluar tecnologías especializadas.

---

## A.2. Arquitectura lógica recomendada por capas

Una arquitectura simple y mantenible puede organizarse con schemas.

```text
postgresql_database
├── raw          # datos crudos cargados desde fuentes externas
├── staging      # datos tipados, normalizados y validados
├── intermediate # transformaciones reutilizables
├── marts        # tablas finales para BI, reporting o ciencia de datos
├── ml           # features, embeddings, predicciones, experimentos
├── metadata     # control de pipelines, calidad, linaje y auditoría
└── sandbox      # análisis exploratorio controlado
```

Crear schemas:

```sql
CREATE SCHEMA IF NOT EXISTS raw;
CREATE SCHEMA IF NOT EXISTS staging;
CREATE SCHEMA IF NOT EXISTS intermediate;
CREATE SCHEMA IF NOT EXISTS marts;
CREATE SCHEMA IF NOT EXISTS ml;
CREATE SCHEMA IF NOT EXISTS metadata;
CREATE SCHEMA IF NOT EXISTS sandbox;
```

Regla práctica:

```text
raw          = conserva lo recibido
staging      = convierte tipos y valida estructura
intermediate = integra y transforma
marts        = expone datos listos para consumo
ml           = sirve features, embeddings y resultados de modelos
metadata     = registra cómo, cuándo y por qué se movieron los datos
sandbox      = permite exploración sin contaminar capas productivas
```

Evita usar solo `public` para todo. En proyectos de datos, separar por schema mejora permisos, trazabilidad, gobierno y lectura del sistema.

---

## A.3. ETL y ELT

### ETL: Extract, Transform, Load

En ETL, los datos se transforman antes de llegar a la base final.

```text
Fuente -> extracción -> transformación externa -> carga en PostgreSQL
```

Útil cuando:

```text
Los datos fuente vienen muy sucios.
La transformación requiere Python, Spark, DuckDB, Polars u otra herramienta externa.
La base destino no debe recibir datos crudos.
Hay restricciones fuertes de privacidad antes de almacenar.
El volumen supera lo que conviene transformar dentro de PostgreSQL.
```

Ejemplo conceptual:

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg://app_user:password@localhost:5432/analytics")

df = pd.read_csv("sales.csv")
df["sale_date"] = pd.to_datetime(df["sale_date"])
df["amount"] = df["amount"].astype(float)
df = df.drop_duplicates(subset=["source_system", "source_id"])

df.to_sql(
    "sales_staging",
    engine,
    schema="staging",
    if_exists="append",
    index=False,
)
```

### ELT: Extract, Load, Transform

En ELT, primero cargas los datos crudos y luego transformas dentro de la base o warehouse.

```text
Fuente -> extracción -> carga cruda -> transformación SQL
```

Útil cuando:

```text
Quieres trazabilidad completa del dato original.
PostgreSQL puede transformar el volumen sin problemas.
Las transformaciones son mayoritariamente SQL.
Quieres reejecutar transformaciones sin volver a extraer la fuente.
Quieres auditar cambios entre raw, staging y marts.
```

Ejemplo de flujo ELT:

```sql
CREATE TABLE raw.sales_csv (
    source_file TEXT NOT NULL,
    loaded_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    raw_payload JSONB NOT NULL
);

CREATE TABLE staging.sales (
    source_system TEXT NOT NULL,
    source_id TEXT NOT NULL,
    sale_date DATE NOT NULL,
    customer_email TEXT,
    amount NUMERIC(12, 2) NOT NULL,
    currency TEXT NOT NULL,
    loaded_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (source_system, source_id)
);

INSERT INTO staging.sales (
    source_system,
    source_id,
    sale_date,
    customer_email,
    amount,
    currency
)
SELECT
    raw_payload ->> 'source_system' AS source_system,
    raw_payload ->> 'source_id' AS source_id,
    (raw_payload ->> 'sale_date')::DATE AS sale_date,
    lower(raw_payload ->> 'customer_email') AS customer_email,
    (raw_payload ->> 'amount')::NUMERIC(12, 2) AS amount,
    raw_payload ->> 'currency' AS currency
FROM raw.sales_csv
WHERE raw_payload ? 'source_id'
  AND raw_payload ? 'sale_date'
  AND raw_payload ? 'amount'
ON CONFLICT (source_system, source_id)
DO UPDATE SET
    sale_date = EXCLUDED.sale_date,
    customer_email = EXCLUDED.customer_email,
    amount = EXCLUDED.amount,
    currency = EXCLUDED.currency,
    loaded_at = now();
```

Regla práctica:

```text
ETL si debes limpiar antes de guardar.
ELT si quieres trazabilidad, reprocesamiento y transformaciones SQL.
En la práctica, muchos sistemas usan una mezcla de ambos.
```

---

## A.4. Diseño de pipelines de datos

Un pipeline de datos mueve información desde fuentes hacia capas analíticas. Puede ser batch, near-real-time o streaming.

Pipeline batch típico:

```text
1. Extraer archivo, API o tabla fuente.
2. Guardar copia cruda.
3. Cargar a tabla raw o staging.
4. Validar estructura y calidad.
5. Transformar a tablas intermedias.
6. Actualizar data marts.
7. Registrar métricas de carga.
8. Notificar éxito o fallo.
```

Tabla de control de ejecuciones:

```sql
CREATE TABLE metadata.pipeline_runs (
    id BIGSERIAL PRIMARY KEY,
    pipeline_name TEXT NOT NULL,
    run_status TEXT NOT NULL CHECK (
        run_status IN ('running', 'success', 'failed', 'skipped')
    ),
    started_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    finished_at TIMESTAMPTZ,
    source_name TEXT,
    source_reference TEXT,
    rows_read BIGINT DEFAULT 0,
    rows_loaded BIGINT DEFAULT 0,
    rows_rejected BIGINT DEFAULT 0,
    error_message TEXT,
    created_by TEXT
);
```

Registrar inicio:

```sql
INSERT INTO metadata.pipeline_runs (
    pipeline_name,
    run_status,
    source_name,
    source_reference,
    created_by
)
VALUES (
    'daily_sales_load',
    'running',
    's3',
    's3://company-data/sales/2026-07-04/sales.csv',
    'airflow'
)
RETURNING id;
```

Registrar éxito:

```sql
UPDATE metadata.pipeline_runs
SET
    run_status = 'success',
    finished_at = now(),
    rows_read = 100000,
    rows_loaded = 99870,
    rows_rejected = 130
WHERE id = 123;
```

Registrar fallo:

```sql
UPDATE metadata.pipeline_runs
SET
    run_status = 'failed',
    finished_at = now(),
    error_message = 'Invalid sale_date in source file'
WHERE id = 123;
```

Buenas prácticas:

```text
Cada ejecución debe tener un run_id.
Cada carga debe ser idempotente.
Cada pipeline debe poder reintentarse.
Cada tabla crítica debe tener conteos y checks mínimos.
Cada error debe quedar registrado con contexto suficiente.
```

---

## A.5. Idempotencia en pipelines

Un pipeline idempotente puede ejecutarse más de una vez sin duplicar ni corromper datos.

Patrones comunes:

### Patrón 1: clave natural + upsert

```sql
INSERT INTO staging.sales (
    source_system,
    source_id,
    sale_date,
    amount
)
VALUES
    ('shopify', 'A-1001', '2026-07-04', 99.90)
ON CONFLICT (source_system, source_id)
DO UPDATE SET
    sale_date = EXCLUDED.sale_date,
    amount = EXCLUDED.amount;
```

### Patrón 2: borrar y recargar partición lógica

```sql
BEGIN;

DELETE FROM staging.sales
WHERE sale_date = DATE '2026-07-04'
  AND source_system = 'shopify';

INSERT INTO staging.sales (
    source_system,
    source_id,
    sale_date,
    amount
)
SELECT
    source_system,
    source_id,
    sale_date,
    amount
FROM raw.sales_shopify_2026_07_04;

COMMIT;
```

### Patrón 3: hash de archivo o lote

```sql
CREATE TABLE metadata.loaded_files (
    id BIGSERIAL PRIMARY KEY,
    source_name TEXT NOT NULL,
    file_path TEXT NOT NULL,
    file_hash TEXT NOT NULL,
    loaded_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (source_name, file_hash)
);
```

Antes de cargar:

```sql
SELECT 1
FROM metadata.loaded_files
WHERE source_name = 'shopify'
  AND file_hash = 'sha256:...';
```

Si ya existe, el pipeline puede saltar la carga o revalidar según política.

---

## A.6. Carga de archivos con `COPY` y `\copy`

PostgreSQL incluye `COPY`, que permite copiar datos entre una tabla y un archivo. `COPY FROM` carga datos desde un archivo hacia una tabla, y `COPY TO` exporta datos desde una tabla hacia un archivo.

Ejemplo con archivo accesible por el servidor PostgreSQL:

```sql
COPY raw.sales_csv (
    source_system,
    source_id,
    sale_date,
    amount,
    currency
)
FROM '/var/lib/postgresql/imports/sales.csv'
WITH (
    FORMAT csv,
    HEADER true,
    DELIMITER ',',
    ENCODING 'UTF8'
);
```

Ejemplo desde el cliente con `psql`:

```sql
\copy raw.sales_csv (
    source_system,
    source_id,
    sale_date,
    amount,
    currency
)
FROM './sales.csv'
WITH (
    FORMAT csv,
    HEADER true,
    DELIMITER ',',
    ENCODING 'UTF8'
);
```

Diferencia práctica:

```text
COPY   = el archivo debe ser accesible desde el servidor PostgreSQL.
\copy  = el archivo se lee desde la máquina cliente que ejecuta psql.
```

Tabla cruda recomendada para CSV:

```sql
CREATE TABLE raw.sales_csv (
    source_file TEXT NOT NULL,
    loaded_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    source_system TEXT,
    source_id TEXT,
    sale_date TEXT,
    customer_email TEXT,
    amount TEXT,
    currency TEXT
);
```

Primero carga todo como texto. Luego transforma a tipos definitivos en `staging`. Esto evita que una fila malformada bloquee toda la interpretación semántica del modelo.

Transformación posterior:

```sql
INSERT INTO staging.sales (
    source_system,
    source_id,
    sale_date,
    customer_email,
    amount,
    currency
)
SELECT
    source_system,
    source_id,
    sale_date::DATE,
    lower(customer_email),
    amount::NUMERIC(12, 2),
    upper(currency)
FROM raw.sales_csv
WHERE source_file = 'sales_2026_07_04.csv'
  AND source_id IS NOT NULL
  AND amount ~ '^[0-9]+(\.[0-9]+)?$';
```

---

## A.7. Manejo de errores de carga

No mezcles filas válidas e inválidas sin registro. Guarda rechazos.

```sql
CREATE TABLE metadata.rejected_rows (
    id BIGSERIAL PRIMARY KEY,
    pipeline_run_id BIGINT REFERENCES metadata.pipeline_runs(id),
    target_table TEXT NOT NULL,
    rejection_reason TEXT NOT NULL,
    raw_payload JSONB NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Ejemplo:

```sql
INSERT INTO metadata.rejected_rows (
    pipeline_run_id,
    target_table,
    rejection_reason,
    raw_payload
)
SELECT
    123,
    'staging.sales',
    'amount is not numeric',
    to_jsonb(raw_sales)
FROM raw.sales_csv AS raw_sales
WHERE source_file = 'sales_2026_07_04.csv'
  AND amount !~ '^[0-9]+(\.[0-9]+)?$';
```

Regla práctica:

```text
No ignores datos rechazados.
No cargues datos inválidos silenciosamente.
No borres raw solo porque staging falló.
```

---

## A.8. Calidad de datos

La calidad de datos debe medirse, no asumirse.

Dimensiones típicas:

```text
Completitud      # faltan datos obligatorios
Unicidad         # duplicados inesperados
Validez          # formato o dominio inválido
Consistencia     # contradicciones entre tablas
Frescura         # datos atrasados
Exactitud        # dato no coincide con fuente confiable
Integridad       # claves foráneas rotas
Distribución     # cambios bruscos en rangos o proporciones
```

Tabla de checks:

```sql
CREATE TABLE metadata.data_quality_checks (
    id BIGSERIAL PRIMARY KEY,
    pipeline_run_id BIGINT REFERENCES metadata.pipeline_runs(id),
    table_name TEXT NOT NULL,
    check_name TEXT NOT NULL,
    check_status TEXT NOT NULL CHECK (check_status IN ('passed', 'failed', 'warning')),
    observed_value NUMERIC,
    expected_value NUMERIC,
    details JSONB,
    checked_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Ejemplo: conteo de nulos.

```sql
INSERT INTO metadata.data_quality_checks (
    pipeline_run_id,
    table_name,
    check_name,
    check_status,
    observed_value,
    expected_value,
    details
)
SELECT
    123,
    'staging.sales',
    'customer_email_null_rate',
    CASE
        WHEN AVG((customer_email IS NULL)::INT) <= 0.05 THEN 'passed'
        ELSE 'failed'
    END,
    AVG((customer_email IS NULL)::INT),
    0.05,
    jsonb_build_object('rule', 'null_rate <= 5%')
FROM staging.sales
WHERE sale_date = DATE '2026-07-04';
```

Ejemplo: duplicados.

```sql
SELECT
    source_system,
    source_id,
    COUNT(*) AS total
FROM staging.sales
GROUP BY source_system, source_id
HAVING COUNT(*) > 1;
```

Ejemplo: frescura.

```sql
SELECT
    MAX(loaded_at) AS last_loaded_at,
    now() - MAX(loaded_at) AS data_age
FROM staging.sales;
```

Buenas prácticas:

```text
Define checks mínimos por tabla crítica.
Registra resultados de checks, no solo logs.
Distingue errores bloqueantes de warnings.
Automatiza alertas para fallos repetidos.
Mantén expectativas por dominio, no solo reglas genéricas.
```

---

## A.9. Gobernanza de datos

La gobernanza de datos define reglas para que los datos sean confiables, localizables, seguros y utilizables.

Componentes mínimos:

```text
Propiedad        # quién responde por el dato
Catálogo         # dónde encontrar tablas, columnas y definiciones
Glosario         # significado común de términos de negocio
Linaje           # de dónde viene el dato y cómo se transformó
Calidad          # reglas, resultados y evolución de checks
Clasificación    # público, interno, confidencial, PII, secreto
Accesos          # quién puede leer, escribir o administrar
Retención        # cuánto tiempo se conserva
Auditoría        # quién accedió o modificó datos críticos
```

Tabla de catálogo simple:

```sql
CREATE TABLE metadata.data_assets (
    id BIGSERIAL PRIMARY KEY,
    schema_name TEXT NOT NULL,
    table_name TEXT NOT NULL,
    asset_type TEXT NOT NULL CHECK (
        asset_type IN ('table', 'view', 'materialized_view', 'external_table')
    ),
    domain_name TEXT,
    owner_team TEXT NOT NULL,
    steward_name TEXT,
    description TEXT,
    classification TEXT NOT NULL CHECK (
        classification IN ('public', 'internal', 'confidential', 'pii', 'secret')
    ),
    retention_days INTEGER,
    is_active BOOLEAN NOT NULL DEFAULT true,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (schema_name, table_name)
);
```

Tabla de columnas:

```sql
CREATE TABLE metadata.data_asset_columns (
    id BIGSERIAL PRIMARY KEY,
    asset_id BIGINT NOT NULL REFERENCES metadata.data_assets(id) ON DELETE CASCADE,
    column_name TEXT NOT NULL,
    data_type TEXT NOT NULL,
    description TEXT,
    is_nullable BOOLEAN,
    is_primary_key BOOLEAN NOT NULL DEFAULT false,
    is_pii BOOLEAN NOT NULL DEFAULT false,
    classification TEXT,
    UNIQUE (asset_id, column_name)
);
```

Ejemplo de registro:

```sql
INSERT INTO metadata.data_assets (
    schema_name,
    table_name,
    asset_type,
    domain_name,
    owner_team,
    steward_name,
    description,
    classification,
    retention_days
)
VALUES (
    'marts',
    'fact_sales',
    'table',
    'sales',
    'data-platform',
    'analytics-lead',
    'Ventas consolidadas listas para reporting.',
    'confidential',
    1825
);
```

Reglas prácticas:

```text
Toda tabla productiva debe tener dueño.
Toda columna sensible debe estar clasificada.
Todo dataset crítico debe tener checks de calidad.
Todo pipeline crítico debe registrar linaje básico.
Todo acceso amplio debe justificarse.
```

---

## A.10. Seguridad, PII y acceso analítico

En contextos de datos, el problema no es solo proteger la base. También hay que evitar que análisis, notebooks o dashboards expongan información sensible.

### Roles por capa

```sql
CREATE ROLE data_engineer;
CREATE ROLE data_scientist;
CREATE ROLE bi_reader;
CREATE ROLE ml_service;
```

Permisos de ejemplo:

```sql
GRANT USAGE ON SCHEMA raw TO data_engineer;
GRANT USAGE ON SCHEMA staging TO data_engineer;
GRANT USAGE ON SCHEMA marts TO data_engineer, data_scientist, bi_reader;

GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA raw TO data_engineer;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA staging TO data_engineer;
GRANT SELECT ON ALL TABLES IN SCHEMA marts TO data_scientist, bi_reader;
```

### Vistas con enmascaramiento

```sql
CREATE VIEW marts.vw_customers_masked AS
SELECT
    customer_id,
    split_part(email, '@', 1) || '@***' AS email_masked,
    country,
    created_at
FROM intermediate.customers;
```

### Hash para unión sin exponer el valor original

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;

SELECT
    encode(digest(lower(email), 'sha256'), 'hex') AS email_hash
FROM intermediate.customers;
```

### Row-Level Security para multi-tenant

```sql
ALTER TABLE marts.fact_sales ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_sales_policy
ON marts.fact_sales
FOR SELECT
USING (tenant_id = current_setting('app.tenant_id')::BIGINT);
```

Regla práctica:

```text
No entregues raw a todos los analistas.
No uses datos personales en notebooks si no es estrictamente necesario.
Prefiere vistas curadas y enmascaradas para consumo amplio.
Separa permisos de ingeniería, ciencia de datos, BI y servicios.
```

---

## A.11. Linaje de datos

El linaje permite responder:

```text
¿De qué fuente viene esta tabla?
¿Qué pipeline la actualizó?
¿Qué tablas alimentan este dashboard?
¿Qué cambio rompió este modelo?
```

Tabla simple de linaje:

```sql
CREATE TABLE metadata.data_lineage (
    id BIGSERIAL PRIMARY KEY,
    pipeline_name TEXT NOT NULL,
    source_asset TEXT NOT NULL,
    target_asset TEXT NOT NULL,
    transformation_type TEXT NOT NULL CHECK (
        transformation_type IN ('copy', 'clean', 'join', 'aggregate', 'feature', 'embedding', 'model_prediction')
    ),
    transformation_reference TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Ejemplo:

```sql
INSERT INTO metadata.data_lineage (
    pipeline_name,
    source_asset,
    target_asset,
    transformation_type,
    transformation_reference
)
VALUES
    ('daily_sales_mart', 'staging.sales', 'intermediate.sales_enriched', 'join', 'sql/intermediate/sales_enriched.sql'),
    ('daily_sales_mart', 'intermediate.sales_enriched', 'marts.fact_sales', 'aggregate', 'sql/marts/fact_sales.sql');
```

Regla práctica:

```text
El linaje no necesita partir perfecto.
Parte registrando fuente, destino, pipeline y archivo SQL.
Luego integra herramientas de catálogo si el proyecto crece.
```

---

## A.12. Data lakes, lakehouses y PostgreSQL

Un datalake almacena datos, normalmente en object storage, en formatos abiertos o semiestructurados.

```text
Ejemplos de almacenamiento:
- Amazon S3
- Google Cloud Storage
- Azure Blob Storage
- MinIO

Ejemplos de formatos:
- CSV
- JSON
- Parquet
- ORC
- Avro

Ejemplos de formatos de tabla lakehouse:
- Apache Iceberg
- Delta Lake
- Apache Hudi
```

Un lakehouse agrega capacidades de tabla, transacciones, snapshots, evolución de schema y lectura desde múltiples motores sobre almacenamiento tipo datalake.

PostgreSQL puede participar en una arquitectura lakehouse como:

```text
Fuente transaccional           # la aplicación escribe en PostgreSQL
Destino de datos curados       # marts pequeños o medianos
Metadata store                 # control de pipelines, calidad y catálogo
Serving layer                  # API, dashboards o aplicaciones internas
Feature store simple           # features tabulares para modelos
Vector store                   # embeddings y búsqueda semántica
```

PostgreSQL no debería usarse como datalake principal cuando:

```text
El volumen crudo crece a decenas o cientos de TB.
Necesitas conservar archivos originales masivos.
Necesitas procesamiento distribuido sobre Parquet.
Necesitas separar cómputo y almacenamiento de forma elástica.
Múltiples motores deben leer los mismos datos a gran escala.
```

Patrón recomendado:

```text
Fuentes operacionales -> datalake raw -> lakehouse silver/gold -> PostgreSQL marts/serving
```

Ejemplo:

```text
1. La app escribe pedidos en PostgreSQL OLTP.
2. Un job CDC o batch exporta cambios a S3/Parquet.
3. Spark, Trino o DuckDB transforman datos lakehouse.
4. Tablas agregadas se publican en PostgreSQL para dashboards o APIs.
5. PostgreSQL conserva metadata, checks, owners y linaje.
```

---

## A.13. Data warehouse con PostgreSQL

Un data warehouse organiza datos históricos, integrados y modelados para análisis.

PostgreSQL puede funcionar bien como warehouse cuando:

```text
El volumen es pequeño o mediano.
Las consultas concurrentes son moderadas.
El equipo domina SQL y PostgreSQL.
Quieres simplicidad operativa.
Los datos analíticos deben integrarse con aplicaciones existentes.
```

Puede quedarse corto cuando:

```text
Hay escaneos frecuentes de muchos TB.
Se requieren cientos de usuarios analíticos concurrentes.
Se necesita MPP nativo.
Se consultan grandes volúmenes columnares.
Se requiere separación elástica entre cómputo y almacenamiento.
```

Capas recomendadas:

```text
raw          # copias fieles
staging      # tipos y limpieza básica
intermediate # joins, reglas reutilizables
marts        # fact/dim o tablas anchas de consumo
```

---

## A.14. Esquema estrella

El esquema estrella es un modelo dimensional donde una tabla de hechos se conecta directamente con tablas de dimensiones.

```text
          dim_customer
              |
dim_date -- fact_sales -- dim_product
              |
          dim_channel
```

### Tabla de hechos

La tabla de hechos contiene eventos medibles.

```sql
CREATE TABLE marts.fact_sales (
    sale_id BIGSERIAL PRIMARY KEY,
    date_key INTEGER NOT NULL,
    customer_key BIGINT NOT NULL,
    product_key BIGINT NOT NULL,
    channel_key BIGINT NOT NULL,
    order_id TEXT NOT NULL,
    quantity INTEGER NOT NULL,
    gross_amount NUMERIC(12, 2) NOT NULL,
    discount_amount NUMERIC(12, 2) NOT NULL DEFAULT 0,
    net_amount NUMERIC(12, 2) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### Dimensión fecha

```sql
CREATE TABLE marts.dim_date (
    date_key INTEGER PRIMARY KEY,
    date_value DATE NOT NULL UNIQUE,
    year INTEGER NOT NULL,
    quarter INTEGER NOT NULL,
    month INTEGER NOT NULL,
    month_name TEXT NOT NULL,
    day_of_month INTEGER NOT NULL,
    day_of_week INTEGER NOT NULL,
    is_weekend BOOLEAN NOT NULL
);
```

### Dimensión cliente

```sql
CREATE TABLE marts.dim_customer (
    customer_key BIGSERIAL PRIMARY KEY,
    customer_id TEXT NOT NULL UNIQUE,
    customer_name TEXT,
    country TEXT,
    segment TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### Dimensión producto

```sql
CREATE TABLE marts.dim_product (
    product_key BIGSERIAL PRIMARY KEY,
    product_id TEXT NOT NULL UNIQUE,
    product_name TEXT NOT NULL,
    category TEXT,
    brand TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### Consulta analítica

```sql
SELECT
    d.year,
    d.month,
    p.category,
    c.segment,
    SUM(f.net_amount) AS revenue,
    SUM(f.quantity) AS units
FROM marts.fact_sales AS f
JOIN marts.dim_date AS d ON d.date_key = f.date_key
JOIN marts.dim_product AS p ON p.product_key = f.product_key
JOIN marts.dim_customer AS c ON c.customer_key = f.customer_key
GROUP BY
    d.year,
    d.month,
    p.category,
    c.segment
ORDER BY
    d.year,
    d.month,
    revenue DESC;
```

Buenas prácticas:

```text
Hechos = eventos, importes, cantidades, métricas.
Dimensiones = contexto de análisis.
Usa surrogate keys en dimensiones.
Mantén nombres claros: fact_*, dim_*.
Documenta el grano de cada tabla de hechos.
```

El grano es crítico. Ejemplo:

```text
fact_sales tiene una fila por línea de pedido.
fact_daily_sales tiene una fila por día, producto y canal.
fact_subscriptions tiene una fila por evento de suscripción.
```

No mezcles granos distintos en la misma tabla de hechos.

---

## A.15. Esquema copo de nieve

El esquema copo de nieve normaliza dimensiones en subdimensiones.

```text
fact_sales -> dim_product -> dim_category
                          -> dim_brand
```

Ejemplo:

```sql
CREATE TABLE marts.dim_category (
    category_key BIGSERIAL PRIMARY KEY,
    category_name TEXT NOT NULL UNIQUE
);

CREATE TABLE marts.dim_brand (
    brand_key BIGSERIAL PRIMARY KEY,
    brand_name TEXT NOT NULL UNIQUE
);

CREATE TABLE marts.dim_product_snowflake (
    product_key BIGSERIAL PRIMARY KEY,
    product_id TEXT NOT NULL UNIQUE,
    product_name TEXT NOT NULL,
    category_key BIGINT NOT NULL REFERENCES marts.dim_category(category_key),
    brand_key BIGINT NOT NULL REFERENCES marts.dim_brand(brand_key)
);
```

Consulta:

```sql
SELECT
    cat.category_name,
    brand.brand_name,
    SUM(f.net_amount) AS revenue
FROM marts.fact_sales AS f
JOIN marts.dim_product_snowflake AS p
    ON p.product_key = f.product_key
JOIN marts.dim_category AS cat
    ON cat.category_key = p.category_key
JOIN marts.dim_brand AS brand
    ON brand.brand_key = p.brand_key
GROUP BY
    cat.category_name,
    brand.brand_name;
```

Comparación:

```text
Estrella:
- Más simple para BI.
- Menos joins.
- Más duplicación en dimensiones.
- Suele ser preferible para consumo analítico.

Copo de nieve:
- Más normalizado.
- Menos duplicación.
- Más joins.
- Útil cuando las jerarquías son grandes o compartidas.
```

Regla práctica:

```text
Prefiere estrella para reporting.
Usa copo de nieve cuando la normalización de dimensiones tenga una ventaja clara.
```

---

## A.16. Slowly Changing Dimensions, SCD

Las dimensiones cambian. Por ejemplo, un cliente cambia de país, segmento o plan.

### SCD Tipo 1

Sobrescribe el valor anterior. Es simple, pero pierde historia.

```sql
UPDATE marts.dim_customer
SET
    segment = 'enterprise',
    country = 'Chile'
WHERE customer_id = 'C-1001';
```

Útil cuando:

```text
No importa conservar historia.
Corriges errores.
El atributo no se analiza históricamente.
```

### SCD Tipo 2

Conserva historia usando vigencia.

```sql
CREATE TABLE marts.dim_customer_scd2 (
    customer_key BIGSERIAL PRIMARY KEY,
    customer_id TEXT NOT NULL,
    customer_name TEXT,
    country TEXT,
    segment TEXT,
    valid_from TIMESTAMPTZ NOT NULL,
    valid_to TIMESTAMPTZ,
    is_current BOOLEAN NOT NULL DEFAULT true,
    row_hash TEXT NOT NULL
);
```

Índice útil:

```sql
CREATE UNIQUE INDEX uq_dim_customer_scd2_current
ON marts.dim_customer_scd2 (customer_id)
WHERE is_current;
```

Cerrar versión anterior:

```sql
UPDATE marts.dim_customer_scd2
SET
    valid_to = now(),
    is_current = false
WHERE customer_id = 'C-1001'
  AND is_current = true;
```

Insertar nueva versión:

```sql
INSERT INTO marts.dim_customer_scd2 (
    customer_id,
    customer_name,
    country,
    segment,
    valid_from,
    valid_to,
    is_current,
    row_hash
)
VALUES (
    'C-1001',
    'Acme Corp',
    'Chile',
    'enterprise',
    now(),
    NULL,
    true,
    encode(digest('Acme Corp|Chile|enterprise', 'sha256'), 'hex')
);
```

Buenas prácticas:

```text
Usa SCD2 solo en dimensiones donde la historia sea analíticamente relevante.
Define qué atributos disparan nueva versión.
Usa row_hash para detectar cambios.
Asegura que exista una sola fila current por entidad.
```

---

## A.17. Data marts

Un data mart es un subconjunto de datos preparado para un área o uso específico.

Ejemplos:

```text
marts.fact_sales             # ventas
marts.customer_360           # vista consolidada de clientes
marts.marketing_attribution  # marketing
marts.ml_churn_features      # features para churn
marts.finance_monthly_pnl    # finanzas
```

Vista materializada para dashboard:

```sql
CREATE MATERIALIZED VIEW marts.mv_monthly_sales AS
SELECT
    d.year,
    d.month,
    p.category,
    SUM(f.net_amount) AS revenue,
    SUM(f.quantity) AS units
FROM marts.fact_sales AS f
JOIN marts.dim_date AS d ON d.date_key = f.date_key
JOIN marts.dim_product AS p ON p.product_key = f.product_key
GROUP BY
    d.year,
    d.month,
    p.category;
```

Índice para refresco concurrente:

```sql
CREATE UNIQUE INDEX uq_mv_monthly_sales
ON marts.mv_monthly_sales (year, month, category);
```

Refresco:

```sql
REFRESH MATERIALIZED VIEW CONCURRENTLY marts.mv_monthly_sales;
```

Regla práctica:

```text
Usa vistas para lógica simple.
Usa vistas materializadas cuando el cálculo sea caro y no necesite tiempo real.
Usa tablas físicas cuando necesites control total de cargas incrementales.
```

---

## A.18. Transformaciones incrementales

No recalcules todo si solo cambió una partición o ventana.

Ejemplo: reconstruir ventas diarias de los últimos 7 días.

```sql
BEGIN;

DELETE FROM marts.daily_sales
WHERE sale_date >= current_date - INTERVAL '7 days';

INSERT INTO marts.daily_sales (
    sale_date,
    product_key,
    channel_key,
    revenue,
    units
)
SELECT
    d.date_value AS sale_date,
    f.product_key,
    f.channel_key,
    SUM(f.net_amount) AS revenue,
    SUM(f.quantity) AS units
FROM marts.fact_sales AS f
JOIN marts.dim_date AS d ON d.date_key = f.date_key
WHERE d.date_value >= current_date - INTERVAL '7 days'
GROUP BY
    d.date_value,
    f.product_key,
    f.channel_key;

COMMIT;
```

Buenas prácticas:

```text
Define una ventana de reproceso.
Incluye late-arriving data si puede llegar tarde.
Usa transacciones para evitar estados intermedios.
Registra filas afectadas.
Valida conteos antes y después.
```

---

## A.19. Particionamiento para datos analíticos

Para tablas grandes de eventos, logs o transacciones, particionar por fecha suele ser natural.

```sql
CREATE TABLE marts.fact_events (
    event_id BIGINT NOT NULL,
    event_date DATE NOT NULL,
    tenant_id BIGINT NOT NULL,
    user_id BIGINT,
    event_name TEXT NOT NULL,
    payload JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (event_id, event_date)
)
PARTITION BY RANGE (event_date);
```

Partición mensual:

```sql
CREATE TABLE marts.fact_events_2026_07
PARTITION OF marts.fact_events
FOR VALUES FROM ('2026-07-01') TO ('2026-08-01');
```

Índice por partición:

```sql
CREATE INDEX idx_fact_events_2026_07_tenant_event
ON marts.fact_events_2026_07 (tenant_id, event_name, event_date);
```

Buenas prácticas:

```text
Particiona por columna usada en filtros.
Crea particiones antes de necesitarlas.
Automatiza creación y retención.
Evita demasiadas particiones pequeñas.
No particiones tablas pequeñas.
```

---

## A.20. Conexión con fuentes externas

### Foreign Data Wrapper hacia otro PostgreSQL

`postgres_fdw` permite acceder a tablas de otro servidor PostgreSQL como si fueran tablas externas.

```sql
CREATE EXTENSION IF NOT EXISTS postgres_fdw;

CREATE SERVER source_pg
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (
    host 'source-db.internal',
    dbname 'source_app',
    port '5432'
);

CREATE USER MAPPING FOR app_user
SERVER source_pg
OPTIONS (
    user 'readonly_user',
    password 'change_this_password'
);

CREATE FOREIGN TABLE raw.remote_orders (
    id BIGINT,
    created_at TIMESTAMPTZ,
    customer_id BIGINT,
    total NUMERIC(12, 2)
)
SERVER source_pg
OPTIONS (
    schema_name 'public',
    table_name 'orders'
);
```

Consulta:

```sql
SELECT *
FROM raw.remote_orders
WHERE created_at >= current_date - INTERVAL '1 day';
```

Usos:

```text
Integración temporal entre bases.
Migraciones.
Consultas exploratorias.
Prototipos de pipelines.
```

Advertencias:

```text
No lo uses como sustituto permanente de un pipeline si hay mucho volumen.
Mide rendimiento.
Filtra lo máximo posible.
Evita joins pesados entre tablas locales y remotas grandes.
```

### Replicación lógica como base para CDC

La replicación lógica usa un modelo de publicación y suscripción.

Publicador:

```sql
CREATE PUBLICATION app_publication
FOR TABLE public.orders, public.customers;
```

Suscriptor:

```sql
CREATE SUBSCRIPTION analytics_subscription
CONNECTION 'host=source-db.internal dbname=app user=replicator password=change_this_password'
PUBLICATION app_publication;
```

Usos:

```text
Copiar cambios de tablas operacionales hacia analítica.
Reducir impacto de lecturas analíticas en la base transaccional.
Mantener una réplica lógica parcial.
```

Advertencias:

```text
No reemplaza backups.
No reemplaza necesariamente una plataforma de streaming.
Requiere monitoreo de lag, slots y conflictos.
Debes entender cómo se propagan deletes, updates y schema changes.
```

---

## A.21. Orquestación con Airflow u otros motores

PostgreSQL no debería ser el único lugar donde vive la lógica de orquestación si el flujo crece. Usa un orquestador para dependencias, retries, schedules y monitoreo.

Herramientas comunes:

```text
Apache Airflow
Dagster
Prefect
dbt Cloud / dbt Core + scheduler
Temporal
Cron simple para proyectos pequeños
```

DAG conceptual:

```text
extract_sales_file
    -> load_raw_sales
    -> validate_raw_sales
    -> transform_staging_sales
    -> build_fact_sales
    -> run_quality_checks
    -> publish_dashboard_tables
```

Buenas prácticas:

```text
Cada tarea debe hacer una cosa.
Las tareas deben ser reintentables.
No pases datasets grandes por el orquestador.
Pasa referencias: rutas, run_id, fechas, particiones.
Registra estado en metadata.pipeline_runs.
Versiona SQL y scripts.
```

Ejemplo conceptual con SQL parametrizado:

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(
    dag_id="daily_sales_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
)
def daily_sales_pipeline():
    @task
    def start_run() -> int:
        # Insertar metadata.pipeline_runs y devolver run_id.
        return 123

    @task
    def load_raw(run_id: int) -> None:
        # Ejecutar COPY o cargar desde object storage.
        ...

    @task
    def transform(run_id: int) -> None:
        # Ejecutar SQL de staging/intermediate/marts.
        ...

    @task
    def quality_checks(run_id: int) -> None:
        # Insertar resultados en metadata.data_quality_checks.
        ...

    run_id = start_run()
    load_raw(run_id)
    transform(run_id)
    quality_checks(run_id)

daily_sales_pipeline()
```

---

## A.22. dbt y transformaciones SQL

dbt encaja especialmente bien en el patrón ELT: cargas datos a una base o warehouse y luego transformas con SQL versionado.

Estructura conceptual:

```text
models/
├── sources.yml
├── staging/
│   └── stg_sales.sql
├── intermediate/
│   └── int_sales_enriched.sql
└── marts/
    └── fact_sales.sql
```

Modelo staging:

```sql
SELECT
    source_system,
    source_id,
    sale_date::DATE AS sale_date,
    lower(customer_email) AS customer_email,
    amount::NUMERIC(12, 2) AS amount,
    upper(currency) AS currency
FROM raw.sales_csv
WHERE source_id IS NOT NULL
```

Modelo mart:

```sql
SELECT
    d.date_key,
    c.customer_key,
    p.product_key,
    s.source_id AS order_id,
    1 AS quantity,
    s.amount AS gross_amount,
    0 AS discount_amount,
    s.amount AS net_amount
FROM {{ ref('stg_sales') }} AS s
JOIN {{ ref('dim_date') }} AS d
    ON d.date_value = s.sale_date
LEFT JOIN {{ ref('dim_customer') }} AS c
    ON c.customer_email = s.customer_email
LEFT JOIN {{ ref('dim_product') }} AS p
    ON p.product_id = s.product_id
```

Buenas prácticas:

```text
Usa modelos pequeños y componibles.
Documenta sources y modelos.
Agrega tests de not_null, unique y relationships.
Usa incremental cuando el volumen lo requiera.
Ejecuta dbt en CI si los modelos son críticos.
```

---

## A.23. Feature engineering en PostgreSQL

PostgreSQL puede servir para construir features tabulares cuando el volumen es manejable.

Tabla de features:

```sql
CREATE TABLE ml.customer_features_daily (
    feature_date DATE NOT NULL,
    customer_id TEXT NOT NULL,
    orders_last_30d INTEGER NOT NULL,
    revenue_last_30d NUMERIC(12, 2) NOT NULL,
    avg_order_value_30d NUMERIC(12, 2),
    days_since_last_order INTEGER,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (feature_date, customer_id)
);
```

Carga incremental:

```sql
INSERT INTO ml.customer_features_daily (
    feature_date,
    customer_id,
    orders_last_30d,
    revenue_last_30d,
    avg_order_value_30d,
    days_since_last_order
)
SELECT
    current_date AS feature_date,
    c.customer_id,
    COUNT(f.sale_id) FILTER (
        WHERE d.date_value >= current_date - INTERVAL '30 days'
    ) AS orders_last_30d,
    COALESCE(SUM(f.net_amount) FILTER (
        WHERE d.date_value >= current_date - INTERVAL '30 days'
    ), 0) AS revenue_last_30d,
    AVG(f.net_amount) FILTER (
        WHERE d.date_value >= current_date - INTERVAL '30 days'
    ) AS avg_order_value_30d,
    current_date - MAX(d.date_value) AS days_since_last_order
FROM marts.dim_customer AS c
LEFT JOIN marts.fact_sales AS f
    ON f.customer_key = c.customer_key
LEFT JOIN marts.dim_date AS d
    ON d.date_key = f.date_key
GROUP BY c.customer_id
ON CONFLICT (feature_date, customer_id)
DO UPDATE SET
    orders_last_30d = EXCLUDED.orders_last_30d,
    revenue_last_30d = EXCLUDED.revenue_last_30d,
    avg_order_value_30d = EXCLUDED.avg_order_value_30d,
    days_since_last_order = EXCLUDED.days_since_last_order,
    created_at = now();
```

Buenas prácticas:

```text
Versiona la definición de features.
Guarda fecha de cálculo.
Evita leakage temporal.
Separa features offline de features online si los requerimientos de latencia difieren.
Registra el dataset usado para entrenar cada modelo.
```

---

## A.24. Registro de experimentos y predicciones

PostgreSQL puede almacenar metadata de experimentos, métricas y predicciones.

```sql
CREATE TABLE ml.model_experiments (
    id BIGSERIAL PRIMARY KEY,
    experiment_name TEXT NOT NULL,
    model_name TEXT NOT NULL,
    model_version TEXT NOT NULL,
    training_dataset TEXT NOT NULL,
    parameters JSONB NOT NULL,
    metrics JSONB NOT NULL,
    artifact_uri TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Predicciones:

```sql
CREATE TABLE ml.model_predictions (
    id BIGSERIAL PRIMARY KEY,
    model_name TEXT NOT NULL,
    model_version TEXT NOT NULL,
    entity_id TEXT NOT NULL,
    prediction_value NUMERIC,
    prediction_label TEXT,
    prediction_score NUMERIC,
    features_snapshot JSONB,
    predicted_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Ejemplo:

```sql
INSERT INTO ml.model_predictions (
    model_name,
    model_version,
    entity_id,
    prediction_label,
    prediction_score,
    features_snapshot
)
VALUES (
    'churn_model',
    '2026-07-04',
    'C-1001',
    'high_risk',
    0.87,
    '{"orders_last_30d": 0, "days_since_last_order": 120}'::JSONB
);
```

Regla práctica:

```text
No guardes solo la predicción.
Guarda también modelo, versión, fecha y snapshot de features o referencia al dataset.
```

---

## A.25. PostgreSQL como base vectorial con pgvector

Una base vectorial almacena embeddings y permite buscar elementos similares. Con la extensión `pgvector`, PostgreSQL puede guardar vectores y ejecutar búsquedas por similitud.

Usos típicos:

```text
Búsqueda semántica.
RAG: Retrieval-Augmented Generation.
Recomendadores.
Deduplicación semántica.
Clasificación asistida por similitud.
Memoria de agentes o asistentes.
```

### Instalación de extensión

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

### Tabla de documentos con embeddings

```sql
CREATE TABLE ml.documents (
    id BIGSERIAL PRIMARY KEY,
    tenant_id BIGINT NOT NULL,
    source_name TEXT NOT NULL,
    source_uri TEXT,
    document_title TEXT,
    chunk_index INTEGER NOT NULL,
    chunk_text TEXT NOT NULL,
    embedding_model TEXT NOT NULL,
    embedding vector(1536) NOT NULL,
    metadata JSONB NOT NULL DEFAULT '{}'::JSONB,
    content_hash TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at TIMESTAMPTZ,
    UNIQUE (tenant_id, source_name, content_hash, chunk_index)
);
```

La dimensión `1536` debe coincidir con el modelo de embeddings usado. Si cambias de modelo y cambia la dimensión, crea otra columna o tabla.

### Inserción de ejemplo

```sql
INSERT INTO ml.documents (
    tenant_id,
    source_name,
    source_uri,
    document_title,
    chunk_index,
    chunk_text,
    embedding_model,
    embedding,
    metadata,
    content_hash
)
VALUES (
    1,
    'docs',
    's3://knowledge-base/postgresql-guide.md',
    'PostgreSQL Guide',
    0,
    'PostgreSQL is an object-relational database system...',
    'text-embedding-model-name',
    '[0.012, -0.003, 0.044]'::vector,
    '{"language": "en", "section": "intro"}',
    'sha256:...'
);
```

El ejemplo usa un vector de 3 valores por brevedad. En una tabla declarada como `vector(1536)`, el vector real debe tener 1536 dimensiones.

---

## A.26. Búsqueda vectorial exacta

Consulta por distancia coseno:

```sql
SELECT
    id,
    document_title,
    chunk_text,
    1 - (embedding <=> '[0.010, -0.004, 0.040]'::vector) AS cosine_similarity
FROM ml.documents
WHERE tenant_id = 1
  AND deleted_at IS NULL
ORDER BY embedding <=> '[0.010, -0.004, 0.040]'::vector
LIMIT 5;
```

Operadores frecuentes en pgvector:

```text
<->  distancia L2
<#>  inner product negativo
<=>  distancia coseno
<+>  distancia L1, si está disponible en la versión instalada
```

Regla práctica:

```text
Usa el operador coherente con cómo fueron entrenados y normalizados tus embeddings.
Para embeddings de texto, coseno suele ser una opción común.
Mide resultados con ejemplos reales, no solo con latencia.
```

---

## A.27. Índices vectoriales: HNSW e IVFFlat

Para tablas pequeñas, búsqueda exacta puede ser suficiente. Para tablas grandes, usa índices aproximados.

### HNSW

```sql
CREATE INDEX idx_documents_embedding_hnsw
ON ml.documents
USING hnsw (embedding vector_cosine_ops);
```

Consulta:

```sql
SELECT
    id,
    chunk_text
FROM ml.documents
WHERE tenant_id = 1
  AND deleted_at IS NULL
ORDER BY embedding <=> '[0.010, -0.004, 0.040]'::vector
LIMIT 10;
```

HNSW suele entregar buena relación entre latencia y recall, pero consume más memoria y puede tardar más en construir.

### IVFFlat

```sql
CREATE INDEX idx_documents_embedding_ivfflat
ON ml.documents
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);
```

Ajuste de probes para la sesión:

```sql
SET ivfflat.probes = 10;
```

IVFFlat suele construir más rápido y usar menos memoria, pero requiere elegir listas/probes y normalmente conviene crearlo después de cargar una cantidad representativa de datos.

Buenas prácticas:

```text
Mide recall y latencia con queries reales.
No indexes embeddings antes de definir la métrica correcta.
Filtra por tenant_id, idioma, fecha o tipo de documento cuando corresponda.
Evalúa particionar si tienes multi-tenancy grande.
Reindexa o reconstruye si cambias de modelo de embeddings.
```

---

## A.28. Búsqueda híbrida: texto + vectores

La búsqueda semántica puede combinarse con filtros relacionales y búsqueda textual.

Agregar columna tsvector:

```sql
ALTER TABLE ml.documents
ADD COLUMN search_vector tsvector
GENERATED ALWAYS AS (
    to_tsvector('spanish', coalesce(document_title, '') || ' ' || chunk_text)
) STORED;
```

Índice textual:

```sql
CREATE INDEX idx_documents_search_vector
ON ml.documents
USING GIN (search_vector);
```

Consulta híbrida simple:

```sql
WITH semantic_search AS (
    SELECT
        id,
        1 - (embedding <=> '[0.010, -0.004, 0.040]'::vector) AS semantic_score
    FROM ml.documents
    WHERE tenant_id = 1
      AND deleted_at IS NULL
    ORDER BY embedding <=> '[0.010, -0.004, 0.040]'::vector
    LIMIT 50
),
keyword_search AS (
    SELECT
        id,
        ts_rank(search_vector, plainto_tsquery('spanish', 'índices particionamiento')) AS keyword_score
    FROM ml.documents
    WHERE tenant_id = 1
      AND deleted_at IS NULL
      AND search_vector @@ plainto_tsquery('spanish', 'índices particionamiento')
)
SELECT
    d.id,
    d.document_title,
    d.chunk_text,
    COALESCE(s.semantic_score, 0) AS semantic_score,
    COALESCE(k.keyword_score, 0) AS keyword_score,
    (COALESCE(s.semantic_score, 0) * 0.7 + COALESCE(k.keyword_score, 0) * 0.3) AS final_score
FROM ml.documents AS d
LEFT JOIN semantic_search AS s ON s.id = d.id
LEFT JOIN keyword_search AS k ON k.id = d.id
WHERE s.id IS NOT NULL OR k.id IS NOT NULL
ORDER BY final_score DESC
LIMIT 10;
```

Buenas prácticas para RAG:

```text
Guarda source_uri y chunk_index.
Guarda embedding_model y versión.
Guarda content_hash para evitar duplicados.
Guarda metadatos filtrables: tenant, idioma, fecha, tipo, permisos.
No mezcles documentos de tenants distintos sin filtros obligatorios.
Implementa borrado lógico o sincronización de deletes.
Evalúa reranking fuera de PostgreSQL si la precisión lo requiere.
```

---

## A.29. Re-embedding y versionado de embeddings

Los embeddings cambian cuando cambias:

```text
Modelo.
Dimensión.
Estrategia de chunking.
Normalización.
Idioma.
Contenido fuente.
```

Tabla para versiones:

```sql
CREATE TABLE ml.embedding_jobs (
    id BIGSERIAL PRIMARY KEY,
    job_name TEXT NOT NULL,
    embedding_model TEXT NOT NULL,
    embedding_dimension INTEGER NOT NULL,
    chunking_strategy TEXT NOT NULL,
    job_status TEXT NOT NULL CHECK (
        job_status IN ('running', 'success', 'failed')
    ),
    started_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    finished_at TIMESTAMPTZ,
    documents_processed BIGINT DEFAULT 0,
    error_message TEXT
);
```

Recomendación:

```text
No sobrescribas embeddings antiguos sin plan.
Crea nueva versión y compara resultados.
Mantén índices separados si conviven modelos diferentes.
Elimina versiones antiguas solo cuando no haya dependencias.
```

---

## A.30. Métricas para búsqueda vectorial

Mide tanto rendimiento como calidad.

Métricas técnicas:

```text
Latencia p50, p95, p99.
Queries por segundo.
Uso de memoria.
Tiempo de construcción de índice.
Tamaño del índice.
Rows examinadas.
```

Métricas de calidad:

```text
Recall@k.
Precision@k.
MRR.
nDCG.
Tasa de respuestas sin contexto útil.
Evaluación humana por muestra.
```

Tabla de evaluación:

```sql
CREATE TABLE ml.retrieval_eval_results (
    id BIGSERIAL PRIMARY KEY,
    eval_name TEXT NOT NULL,
    query_text TEXT NOT NULL,
    expected_document_id BIGINT,
    retrieved_document_ids BIGINT[] NOT NULL,
    rank_found INTEGER,
    metric_values JSONB NOT NULL,
    evaluated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Regla práctica:

```text
No optimices solo latencia.
Una búsqueda rápida que recupera mal contexto degrada todo el sistema RAG.
```

---

## A.31. Observabilidad para datos y analítica

Además de observar PostgreSQL como base, observa los datos como producto.

Mínimos por pipeline:

```text
Última ejecución exitosa.
Duración.
Filas leídas, cargadas y rechazadas.
Freshness.
Checks fallidos.
Tablas modificadas.
Volumen por partición.
Errores por fuente.
```

Consulta de freshness:

```sql
SELECT
    pipeline_name,
    MAX(finished_at) AS last_success
FROM metadata.pipeline_runs
WHERE run_status = 'success'
GROUP BY pipeline_name
ORDER BY last_success ASC;
```

Consulta de fallos recientes:

```sql
SELECT
    pipeline_name,
    source_name,
    source_reference,
    error_message,
    started_at,
    finished_at
FROM metadata.pipeline_runs
WHERE run_status = 'failed'
  AND started_at >= now() - INTERVAL '7 days'
ORDER BY started_at DESC;
```

Consulta de checks fallidos:

```sql
SELECT
    table_name,
    check_name,
    observed_value,
    expected_value,
    details,
    checked_at
FROM metadata.data_quality_checks
WHERE check_status = 'failed'
  AND checked_at >= now() - INTERVAL '7 days'
ORDER BY checked_at DESC;
```

---

## A.32. Buenas prácticas para notebooks y ciencia de datos

Los notebooks son útiles para exploración, pero peligrosos como sistema productivo.

Reglas:

```text
No uses credenciales personales con permisos amplios.
No ejecutes queries sin LIMIT durante exploración.
No exportes PII localmente sin justificación.
No escribas directo en marts productivos desde notebooks.
No conviertas un notebook en pipeline crítico sin refactorizar.
```

Patrón recomendado:

```text
notebook exploratorio -> script versionado -> pipeline orquestado -> tabla de salida controlada
```

Crear schema sandbox por usuario o equipo:

```sql
CREATE SCHEMA IF NOT EXISTS sandbox_felipe;

GRANT USAGE, CREATE ON SCHEMA sandbox_felipe TO data_scientist;
```

Vista segura para análisis:

```sql
CREATE VIEW marts.vw_sales_for_ds AS
SELECT
    sale_id,
    date_key,
    product_key,
    channel_key,
    quantity,
    net_amount
FROM marts.fact_sales;
```

Evita entregar tablas con PII si el análisis no lo requiere.

---

## A.33. Rendimiento analítico

Buenas prácticas:

```text
Modela primero, indexa después.
Usa EXPLAIN y EXPLAIN ANALYZE.
Agrega índices sobre claves de joins y filtros frecuentes.
Usa índices BRIN para tablas grandes ordenadas por fecha.
Usa particionamiento cuando el volumen y los filtros lo justifiquen.
Evita SELECT * en tablas anchas.
Materializa agregaciones costosas.
No dejes dashboards golpeando raw.
```

Índices típicos para esquema estrella:

```sql
CREATE INDEX idx_fact_sales_date_key
ON marts.fact_sales (date_key);

CREATE INDEX idx_fact_sales_customer_key
ON marts.fact_sales (customer_key);

CREATE INDEX idx_fact_sales_product_key
ON marts.fact_sales (product_key);

CREATE INDEX idx_fact_sales_channel_key
ON marts.fact_sales (channel_key);
```

Índice compuesto si filtras por fecha y producto:

```sql
CREATE INDEX idx_fact_sales_date_product
ON marts.fact_sales (date_key, product_key);
```

BRIN para tabla enorme por fecha:

```sql
CREATE INDEX idx_fact_events_event_date_brin
ON marts.fact_events
USING BRIN (event_date);
```

---

## A.34. Antipatrones comunes

```text
Usar public para todo.
Cargar CSV directo a tablas finales.
No registrar run_id.
No guardar datos rechazados.
No distinguir raw de staging.
Hacer dashboards sobre tablas raw.
Usar SELECT * en procesos productivos.
No definir grano de tablas de hechos.
Mezclar métricas de distintos granos.
No tener owner de tablas.
No clasificar PII.
Dar acceso directo a raw a toda la organización.
Usar PostgreSQL como datalake de archivos masivos.
Crear índices sin medir.
No probar restores de datos analíticos.
No versionar SQL de transformaciones.
Sobrescribir embeddings sin versionar modelo.
No medir calidad de recuperación en RAG.
```

---

## A.35. Checklist para pipelines PostgreSQL

```text
[ ] ¿Existe un run_id por ejecución?
[ ] ¿La carga es idempotente?
[ ] ¿Hay tabla raw o copia de fuente?
[ ] ¿Hay validación antes de publicar datos?
[ ] ¿Se registran filas rechazadas?
[ ] ¿Se registran conteos de filas?
[ ] ¿Hay checks de calidad mínimos?
[ ] ¿Hay owner de cada tabla final?
[ ] ¿Hay linaje fuente -> destino?
[ ] ¿Los permisos están separados por rol?
[ ] ¿La tabla final tiene grano documentado?
[ ] ¿La estrategia incremental está definida?
[ ] ¿Hay plan para late-arriving data?
[ ] ¿Hay monitoreo de freshness?
[ ] ¿Hay rollback o reproceso?
```

---

## A.36. Checklist para data warehouse

```text
[ ] ¿Las tablas de hechos tienen grano explícito?
[ ] ¿Las dimensiones tienen surrogate keys?
[ ] ¿Las claves naturales están conservadas?
[ ] ¿Las métricas tienen definición de negocio?
[ ] ¿Hay calendario/dim_date consistente?
[ ] ¿Se definió SCD Tipo 1 o Tipo 2 cuando aplica?
[ ] ¿Los dashboards consumen marts, no raw?
[ ] ¿Las vistas materializadas tienen política de refresh?
[ ] ¿Hay índices para joins principales?
[ ] ¿Hay tests de integridad referencial?
```

---

## A.37. Checklist para bases vectoriales en PostgreSQL

```text
[ ] ¿La dimensión del vector coincide con el modelo?
[ ] ¿Se guarda embedding_model?
[ ] ¿Se guarda versión de chunking?
[ ] ¿Se guarda source_uri?
[ ] ¿Se guarda content_hash?
[ ] ¿Se filtra por tenant/permisos antes de recuperar contexto?
[ ] ¿Se eligió métrica: coseno, L2 o inner product?
[ ] ¿Se midió recall@k?
[ ] ¿Hay estrategia de re-embedding?
[ ] ¿Hay borrado lógico o sincronización de deletes?
[ ] ¿Hay búsqueda híbrida si keywords importan?
[ ] ¿Hay evaluación humana o golden set?
```

---

## A.38. Ejemplo de arquitectura completa

```text
Aplicación FastAPI
    -> PostgreSQL OLTP
        -> publicación lógica / batch export
            -> raw en datalake
                -> transformaciones lakehouse
                    -> marts en PostgreSQL
                        -> dashboards / APIs / notebooks

Documentos internos
    -> extracción
        -> chunking
            -> embeddings
                -> PostgreSQL + pgvector
                    -> búsqueda semántica / RAG
```

Componentes PostgreSQL:

```text
public/app schemas     # sistema transaccional
raw                    # cargas crudas
staging                # datos tipados
intermediate           # reglas reutilizables
marts                  # consumo analítico
ml                     # features, embeddings, predicciones
metadata               # control, calidad, linaje, catálogo
```

Regla final:

```text
PostgreSQL funciona muy bien como núcleo confiable para datos estructurados, marts, metadata y búsqueda vectorial integrada.
No lo conviertas por fuerza en motor distribuido, datalake masivo o plataforma única para todos los problemas.
Diseña la arquitectura según volumen, latencia, gobierno, costo y criticidad.
```

---

## A.39. Referencias técnicas sugeridas

- PostgreSQL Documentation: `COPY`.
- PostgreSQL Documentation: Materialized Views.
- PostgreSQL Documentation: Logical Replication.
- PostgreSQL Documentation: `postgres_fdw`.
- PostgreSQL Documentation: Indexes, Partitioning, Row-Level Security.
- pgvector: documentación del proyecto y ejemplos de índices HNSW/IVFFlat.
- Apache Airflow: documentación sobre ETL/ELT y DAGs.
- Apache Iceberg: especificación de formato, snapshots y evolución de schema.
- Microsoft Learn: modelado dimensional y esquema estrella.
- OpenMetadata: glosarios, gobierno, clasificación y metadata.
- dbt: transformaciones SQL, modelos, tests, documentación y linaje.
