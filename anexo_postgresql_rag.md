# Anexo II: PostgreSQL para RAG, búsqueda vectorial y documentos JSONB

## 1. Objetivo

Este anexo complementa la guía principal de PostgreSQL con un enfoque aplicado a sistemas **RAG** (*Retrieval-Augmented Generation*). Su objetivo es mostrar cómo usar PostgreSQL como base documental, almacén de metadatos, motor de búsqueda textual y motor de búsqueda vectorial usando `pgvector`.

La guía está pensada para ingenieros de datos, científicos de datos, desarrolladores backend y equipos que quieren construir un sistema RAG reproducible, auditable y mantenible sin introducir una base vectorial separada desde el primer día.

El anexo cubre:

```text
1. Qué es RAG y cuándo usarlo.
2. Cómo modelar documentos, chunks, embeddings y metadatos en PostgreSQL.
3. Cómo usar JSONB para metadatos y contenido semiestructurado.
4. Cómo crear índices de texto, JSONB y vectores.
5. Cómo hacer búsqueda vectorial, textual e híbrida.
6. Cómo segmentar documentos: tamaño fijo, párrafos, secciones, semántica y jerarquía.
7. Cómo leer TXT, Markdown, DOCX, PDF, HTML, CSV y otros formatos.
8. Cómo usar BERT, BETO y modelos modernos de embeddings.
9. Cómo evaluar calidad de recuperación.
10. Cómo operar un sistema RAG en producción.
```

---

## 2. Qué es RAG

RAG es una arquitectura donde un modelo generativo no responde solo con lo que tiene en sus pesos internos. Antes de generar una respuesta, el sistema busca información relevante en una base de conocimiento externa y entrega esos fragmentos al modelo como contexto.

Flujo básico:

```text
Pregunta del usuario
    ↓
Normalización de la pregunta
    ↓
Embedding de la pregunta
    ↓
Búsqueda en PostgreSQL
    ↓
Recuperación de chunks relevantes
    ↓
Reordenamiento opcional
    ↓
Construcción del prompt con contexto
    ↓
Respuesta del LLM con citas o referencias
```

RAG es útil cuando:

```text
[ ] La información cambia con frecuencia.
[ ] Necesitas responder sobre documentos internos.
[ ] Necesitas citar fuentes.
[ ] No quieres reentrenar un modelo por cada cambio documental.
[ ] Necesitas separar conocimiento de razonamiento.
[ ] Necesitas controlar permisos por usuario, área, tenant o documento.
```

RAG no reemplaza:

```text
[ ] Una buena limpieza documental.
[ ] Una taxonomía clara.
[ ] Evaluación de recuperación.
[ ] Control de acceso.
[ ] Gobierno de datos.
[ ] Revisión humana en dominios críticos.
```

---

## 3. Arquitectura recomendada con PostgreSQL

Una arquitectura mínima puede tener estas capas:

```text
raw_documents       # documentos originales o referencias a ellos
processed_documents # texto extraído y normalizado
chunks              # fragmentos indexables
embeddings          # vectores asociados a chunks
retrieval_logs      # auditoría de consultas y resultados
feedback            # evaluación humana o automática
```

En proyectos pequeños, todas estas capas pueden vivir en una sola base PostgreSQL. En proyectos grandes, PostgreSQL puede actuar como índice semántico y catálogo operativo, mientras los binarios originales viven en S3, GCS, Azure Blob, MinIO, un datalake o un gestor documental.

Regla práctica:

```text
PostgreSQL debe guardar texto, metadatos, vectores, trazabilidad y relaciones.
No necesariamente debe guardar todos los binarios pesados.
```

---

## 4. Extensiones recomendadas

Activa extensiones según necesidad:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS unaccent;
CREATE EXTENSION IF NOT EXISTS pg_trgm;
```

Uso:

```text
vector    # tipo vector y búsqueda de similitud con pgvector
pgcrypto  # gen_random_uuid()
unaccent  # normalización para búsqueda textual
pg_trgm   # similitud textual por trigramas
```

Instalación conceptual de `pgvector`:

```bash
# Debian/Ubuntu, ejemplo conceptual. Ajusta versión de PostgreSQL según tu ambiente.
sudo apt install postgresql-18-pgvector
```

En Docker, una opción habitual es usar una imagen que ya incluya `pgvector`, o construir una propia a partir de PostgreSQL.

---

## 5. Modelo de datos base para RAG

### 5.1 Tabla de documentos

```sql
CREATE TABLE rag_documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_uri TEXT NOT NULL,
    source_type TEXT NOT NULL CHECK (source_type IN (
        'txt',
        'markdown',
        'docx',
        'pdf',
        'html',
        'csv',
        'xlsx',
        'pptx',
        'other'
    )),
    title TEXT,
    language TEXT NOT NULL DEFAULT 'es',
    checksum TEXT NOT NULL,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    ingested_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at TIMESTAMPTZ,
    UNIQUE (source_uri, checksum)
);
```

`checksum` permite detectar documentos repetidos o versiones ya procesadas.

Ejemplo de metadatos:

```json
{
    "department": "legal",
    "project": "rpg-book-reaper",
    "confidentiality": "internal",
    "owner": "data-team",
    "version": "2026-07-04",
    "tags": ["contratos", "rag", "postgresql"]
}
```

### 5.2 Tabla de chunks

La dimensión del vector depende del modelo de embedding. Por ejemplo:

```text
384   # all-MiniLM-L6-v2 y modelos pequeños similares
768   # BERT base, BETO base, varios modelos E5 base
1024  # algunos modelos grandes
1536  # modelos comerciales frecuentes
3072  # modelos comerciales grandes
```

Ejemplo con dimensión 768:

```sql
CREATE TABLE rag_chunks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id UUID NOT NULL REFERENCES rag_documents(id) ON DELETE CASCADE,
    parent_chunk_id UUID REFERENCES rag_chunks(id) ON DELETE CASCADE,
    chunk_index INTEGER NOT NULL,
    chunk_level INTEGER NOT NULL DEFAULT 0,
    chunk_strategy TEXT NOT NULL,
    content TEXT NOT NULL,
    content_hash TEXT NOT NULL,
    word_count INTEGER NOT NULL,
    char_count INTEGER NOT NULL,
    token_count INTEGER,
    section_title TEXT,
    page_number INTEGER,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    embedding vector(768),
    search_vector tsvector GENERATED ALWAYS AS (
        setweight(to_tsvector('spanish', coalesce(section_title, '')), 'A') ||
        setweight(to_tsvector('spanish', coalesce(content, '')), 'B')
    ) STORED,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (document_id, chunk_index, chunk_level, content_hash)
);
```

Notas:

```text
parent_chunk_id permite chunking jerárquico.
chunk_level permite distinguir chunks padre, sección, párrafo o fragmento final.
chunk_strategy registra cómo se segmentó el texto.
metadata permite guardar page, heading, author, tags, permisos, etc.
embedding puede ser NULL mientras el chunk espera vectorización.
search_vector permite búsqueda full-text nativa de PostgreSQL.
```

### 5.3 Tabla de consultas y auditoría

```sql
CREATE TABLE rag_retrieval_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id TEXT,
    query_text TEXT NOT NULL,
    query_metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    retrieved_chunk_ids UUID[] NOT NULL DEFAULT '{}',
    latency_ms INTEGER,
    model_name TEXT,
    embedding_model_name TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Esto permite evaluar:

```text
Qué preguntas llegan.
Qué chunks se recuperan.
Qué filtros se aplican.
Qué latencia tiene el retrieval.
Qué modelo de embedding estaba vigente.
```

---

## 6. Índices recomendados

### 6.1 Índices generales

```sql
CREATE INDEX idx_rag_documents_source_type
ON rag_documents (source_type)
WHERE deleted_at IS NULL;

CREATE INDEX idx_rag_documents_language
ON rag_documents (language)
WHERE deleted_at IS NULL;

CREATE INDEX idx_rag_chunks_document_id
ON rag_chunks (document_id);

CREATE INDEX idx_rag_chunks_parent_chunk_id
ON rag_chunks (parent_chunk_id);

CREATE INDEX idx_rag_chunks_page_number
ON rag_chunks (page_number)
WHERE page_number IS NOT NULL;
```

### 6.2 Índice full-text

```sql
CREATE INDEX idx_rag_chunks_search_vector
ON rag_chunks
USING GIN (search_vector);
```

Uso:

```sql
SELECT
    id,
    section_title,
    content,
    ts_rank_cd(search_vector, query) AS rank
FROM
    rag_chunks,
    plainto_tsquery('spanish', 'política de retención documental') query
WHERE
    search_vector @@ query
ORDER BY
    rank DESC
LIMIT 10;
```

### 6.3 Índices JSONB

Índice general sobre metadatos:

```sql
CREATE INDEX idx_rag_documents_metadata_gin
ON rag_documents
USING GIN (metadata);

CREATE INDEX idx_rag_chunks_metadata_gin
ON rag_chunks
USING GIN (metadata);
```

Índice más compacto para búsquedas de contención:

```sql
CREATE INDEX idx_rag_chunks_metadata_path_ops
ON rag_chunks
USING GIN (metadata jsonb_path_ops);
```

Ejemplo de filtro:

```sql
SELECT id, content
FROM rag_chunks
WHERE metadata @> '{"department": "legal"}'::jsonb
LIMIT 20;
```

Índice por expresión para filtros frecuentes:

```sql
CREATE INDEX idx_rag_chunks_metadata_department
ON rag_chunks ((metadata ->> 'department'));

CREATE INDEX idx_rag_chunks_metadata_confidentiality
ON rag_chunks ((metadata ->> 'confidentiality'));
```

Regla práctica:

```text
Usa JSONB para metadatos flexibles.
Usa columnas normales para campos obligatorios o muy consultados.
Usa índices por expresión para claves JSONB que filtras con frecuencia.
```

### 6.4 Índices vectoriales con HNSW

Para similitud coseno:

```sql
CREATE INDEX idx_rag_chunks_embedding_hnsw_cosine
ON rag_chunks
USING hnsw (embedding vector_cosine_ops)
WHERE embedding IS NOT NULL;
```

Para producto interno:

```sql
CREATE INDEX idx_rag_chunks_embedding_hnsw_ip
ON rag_chunks
USING hnsw (embedding vector_ip_ops)
WHERE embedding IS NOT NULL;
```

Para distancia L2:

```sql
CREATE INDEX idx_rag_chunks_embedding_hnsw_l2
ON rag_chunks
USING hnsw (embedding vector_l2_ops)
WHERE embedding IS NOT NULL;
```

Recomendación:

```text
Usa vector_cosine_ops cuando tus embeddings estén pensados para similitud coseno.
Usa vector_ip_ops si el modelo recomienda producto interno y los vectores están normalizados.
Usa vector_l2_ops si el modelo o benchmark local demuestra mejor comportamiento con L2.
No crees los tres índices si solo usas una métrica.
```

Ajuste por consulta:

```sql
BEGIN;
SET LOCAL hnsw.ef_search = 100;

SELECT id, content, embedding <=> '[0.1, 0.2, 0.3]'::vector AS distance
FROM rag_chunks
WHERE embedding IS NOT NULL
ORDER BY embedding <=> '[0.1, 0.2, 0.3]'::vector
LIMIT 10;

COMMIT;
```

### 6.5 Índices vectoriales con IVFFlat

IVFFlat requiere que ya exista una cantidad representativa de datos antes de crear el índice.

```sql
CREATE INDEX idx_rag_chunks_embedding_ivfflat_cosine
ON rag_chunks
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100)
WHERE embedding IS NOT NULL;
```

Ajuste por consulta:

```sql
BEGIN;
SET LOCAL ivfflat.probes = 10;

SELECT id, content, embedding <=> '[0.1, 0.2, 0.3]'::vector AS distance
FROM rag_chunks
WHERE embedding IS NOT NULL
ORDER BY embedding <=> '[0.1, 0.2, 0.3]'::vector
LIMIT 10;

COMMIT;
```

Uso recomendado:

```text
HNSW: mejor punto de partida si necesitas buena relación velocidad/recall y tienes memoria suficiente.
IVFFlat: útil para cargas grandes donde quieres menor memoria y construcción más rápida, aceptando más ajuste manual.
Búsqueda exacta sin índice ANN: útil para tablas pequeñas, debugging y evaluación de recall.
```

---

## 7. Búsqueda vectorial básica

Una búsqueda vectorial responde: “dado el embedding de la pregunta, dame los chunks más cercanos”.

```sql
SELECT
    c.id,
    c.document_id,
    d.title,
    c.section_title,
    c.content,
    c.embedding <=> :query_embedding AS distance
FROM rag_chunks c
JOIN rag_documents d ON d.id = c.document_id
WHERE
    d.deleted_at IS NULL
    AND c.embedding IS NOT NULL
ORDER BY
    c.embedding <=> :query_embedding
LIMIT 8;
```

Con filtro de metadatos:

```sql
SELECT
    c.id,
    d.title,
    c.content,
    c.embedding <=> :query_embedding AS distance
FROM rag_chunks c
JOIN rag_documents d ON d.id = c.document_id
WHERE
    d.deleted_at IS NULL
    AND c.embedding IS NOT NULL
    AND d.language = 'es'
    AND d.metadata @> '{"department": "legal"}'::jsonb
ORDER BY
    c.embedding <=> :query_embedding
LIMIT 8;
```

Regla práctica:

```text
Filtra por permisos, tenant, idioma y dominio antes de entregar contexto al LLM.
No confíes en que el prompt impedirá fugas de información.
```

---

## 8. Búsqueda textual con PostgreSQL

La búsqueda textual es útil cuando:

```text
[ ] El usuario menciona códigos exactos.
[ ] Hay nombres propios.
[ ] Hay IDs, leyes, cláusulas, tickets o tablas.
[ ] La pregunta depende de coincidencia literal.
[ ] Los embeddings no capturan bien términos técnicos.
```

Ejemplo:

```sql
SELECT
    c.id,
    d.title,
    c.section_title,
    c.content,
    ts_rank_cd(c.search_vector, query) AS text_rank
FROM
    rag_chunks c
JOIN rag_documents d ON d.id = c.document_id,
    plainto_tsquery('spanish', :query_text) query
WHERE
    d.deleted_at IS NULL
    AND c.search_vector @@ query
ORDER BY
    text_rank DESC
LIMIT 10;
```

Con `websearch_to_tsquery`:

```sql
SELECT
    c.id,
    c.content,
    ts_rank_cd(c.search_vector, query) AS text_rank
FROM
    rag_chunks c,
    websearch_to_tsquery('spanish', :query_text) query
WHERE
    c.search_vector @@ query
ORDER BY
    text_rank DESC
LIMIT 10;
```

`websearch_to_tsquery` suele ser cómodo para búsquedas escritas por usuarios, porque acepta una sintaxis más parecida a buscadores web.

---

## 9. Búsqueda híbrida

La búsqueda híbrida combina:

```text
Búsqueda vectorial  # similitud semántica
Búsqueda textual    # coincidencia léxica
Filtros relacionales # permisos, tenant, fechas, idioma, tipo documental
```

### 9.1 Híbrida simple por score combinado

```sql
WITH vector_results AS (
    SELECT
        c.id,
        1 - (c.embedding <=> :query_embedding) AS vector_score
    FROM rag_chunks c
    JOIN rag_documents d ON d.id = c.document_id
    WHERE
        d.deleted_at IS NULL
        AND c.embedding IS NOT NULL
    ORDER BY c.embedding <=> :query_embedding
    LIMIT 50
),
text_results AS (
    SELECT
        c.id,
        ts_rank_cd(c.search_vector, query) AS text_score
    FROM
        rag_chunks c,
        plainto_tsquery('spanish', :query_text) query
    JOIN rag_documents d ON d.id = c.document_id
    WHERE
        d.deleted_at IS NULL
        AND c.search_vector @@ query
    ORDER BY text_score DESC
    LIMIT 50
),
combined AS (
    SELECT
        COALESCE(v.id, t.id) AS id,
        COALESCE(v.vector_score, 0) AS vector_score,
        COALESCE(t.text_score, 0) AS text_score,
        (0.70 * COALESCE(v.vector_score, 0)) +
        (0.30 * COALESCE(t.text_score, 0)) AS final_score
    FROM vector_results v
    FULL OUTER JOIN text_results t ON t.id = v.id
)
SELECT
    c.id,
    d.title,
    c.section_title,
    c.content,
    combined.vector_score,
    combined.text_score,
    combined.final_score
FROM combined
JOIN rag_chunks c ON c.id = combined.id
JOIN rag_documents d ON d.id = c.document_id
ORDER BY combined.final_score DESC
LIMIT 10;
```

Advertencia:

```text
Los scores vectoriales y textuales no siempre están en la misma escala.
Para producción, evalúa normalización, percentiles o Reciprocal Rank Fusion.
```

### 9.2 Híbrida por Reciprocal Rank Fusion, RRF

RRF combina rankings en vez de combinar scores crudos.

```sql
WITH vector_ranked AS (
    SELECT
        c.id,
        row_number() OVER (ORDER BY c.embedding <=> :query_embedding) AS rank_vector
    FROM rag_chunks c
    JOIN rag_documents d ON d.id = c.document_id
    WHERE
        d.deleted_at IS NULL
        AND c.embedding IS NOT NULL
    ORDER BY c.embedding <=> :query_embedding
    LIMIT 100
),
text_ranked AS (
    SELECT
        c.id,
        row_number() OVER (ORDER BY ts_rank_cd(c.search_vector, query) DESC) AS rank_text
    FROM
        rag_chunks c,
        plainto_tsquery('spanish', :query_text) query
    JOIN rag_documents d ON d.id = c.document_id
    WHERE
        d.deleted_at IS NULL
        AND c.search_vector @@ query
    ORDER BY ts_rank_cd(c.search_vector, query) DESC
    LIMIT 100
),
combined AS (
    SELECT
        COALESCE(v.id, t.id) AS id,
        COALESCE(1.0 / (60 + v.rank_vector), 0) AS rrf_vector,
        COALESCE(1.0 / (60 + t.rank_text), 0) AS rrf_text
    FROM vector_ranked v
    FULL OUTER JOIN text_ranked t ON t.id = v.id
)
SELECT
    c.id,
    d.title,
    c.content,
    (combined.rrf_vector + combined.rrf_text) AS rrf_score
FROM combined
JOIN rag_chunks c ON c.id = combined.id
JOIN rag_documents d ON d.id = c.document_id
ORDER BY rrf_score DESC
LIMIT 10;
```

RRF suele ser más estable que una suma lineal de scores cuando cada motor produce escalas distintas.

---

## 10. JSONB para documentos y metadatos

En RAG, JSONB sirve para guardar metadatos variables sin alterar el esquema por cada fuente documental.

Casos adecuados:

```text
[ ] Tags variables.
[ ] Propiedades por tipo de documento.
[ ] Información de extracción.
[ ] Datos de OCR.
[ ] Coordenadas de página.
[ ] Jerarquía de títulos.
[ ] Permisos complementarios.
[ ] Señales de calidad documental.
```

Ejemplo de metadata por chunk:

```json
{
    "source": "manual_tecnico.pdf",
    "page": 17,
    "heading_path": ["Instalación", "PostgreSQL", "pgvector"],
    "parser": "pymupdf",
    "ocr": false,
    "tables_detected": 2,
    "contains_code": true,
    "tenant_id": "org_123",
    "allowed_roles": ["data_engineer", "ml_engineer"]
}
```

Consulta por JSONB:

```sql
SELECT id, content
FROM rag_chunks
WHERE metadata @> '{"contains_code": true}'::jsonb;
```

Consulta por ruta JSON:

```sql
SELECT id, content
FROM rag_chunks
WHERE metadata #>> '{heading_path,0}' = 'Instalación';
```

Consulta por rol permitido:

```sql
SELECT id, content
FROM rag_chunks
WHERE metadata -> 'allowed_roles' ? 'data_engineer';
```

Buena práctica:

```text
No guardes todo como JSONB.
Si un campo es obligatorio, relacional o crítico para filtros, conviértelo en columna.
Si un campo cambia por tipo documental, JSONB es adecuado.
```

---

## 11. Estrategias de segmentación de documentos

La segmentación, o *chunking*, determina qué unidades recuperará el sistema. Es una de las decisiones más importantes en RAG.

No existe un estándar universal de “300”. Según la herramienta, el tamaño puede medirse en:

```text
Caracteres.
Palabras.
Tokens.
Sentencias.
Párrafos.
Secciones.
Nodos jerárquicos.
```

Como punto de partida para documentos generales:

```text
300–500 palabras por chunk.
800–1.500 tokens por chunk.
1.500–3.000 caracteres por chunk.
10–20% de overlap.
```

Pero esos valores deben ajustarse según:

```text
Tipo de documento.
Modelo de embedding.
Modelo generativo.
Longitud de contexto disponible.
Necesidad de citas precisas.
Complejidad de la pregunta.
Dominio técnico.
Presencia de tablas o código.
```

---

## 12. Segmentación por tamaño fijo

Es la técnica más simple. Divide el texto cada N caracteres, palabras o tokens.

Ventajas:

```text
Simple.
Rápida.
Fácil de reproducir.
Funciona como baseline.
```

Desventajas:

```text
Puede cortar frases o ideas.
Puede separar título y contenido.
Puede romper tablas, listas y código.
Puede perder contexto.
```

### 12.1 Por caracteres

```python
from __future__ import annotations


def chunk_by_characters(
    text: str,
    chunk_size: int = 1800,
    overlap: int = 200,
) -> list[str]:
    if chunk_size <= 0:
        raise ValueError("chunk_size debe ser mayor que 0")

    if overlap < 0 or overlap >= chunk_size:
        raise ValueError("overlap debe ser >= 0 y menor que chunk_size")

    chunks: list[str] = []
    start = 0

    while start < len(text):
        end = start + chunk_size
        chunk = text[start:end].strip()

        if chunk:
            chunks.append(chunk)

        start = end - overlap

    return chunks
```

### 12.2 Por palabras

```python
from __future__ import annotations


def chunk_by_words(
    text: str,
    chunk_size: int = 300,
    overlap: int = 50,
) -> list[str]:
    words = text.split()

    if chunk_size <= 0:
        raise ValueError("chunk_size debe ser mayor que 0")

    if overlap < 0 or overlap >= chunk_size:
        raise ValueError("overlap debe ser >= 0 y menor que chunk_size")

    chunks: list[str] = []
    start = 0

    while start < len(words):
        end = start + chunk_size
        chunk = " ".join(words[start:end]).strip()

        if chunk:
            chunks.append(chunk)

        start = end - overlap

    return chunks
```

### 12.3 Por tokens con tokenizer Hugging Face

```python
from __future__ import annotations

from transformers import AutoTokenizer


def chunk_by_tokens(
    text: str,
    model_name: str = "intfloat/multilingual-e5-base",
    chunk_size: int = 512,
    overlap: int = 64,
) -> list[str]:
    tokenizer = AutoTokenizer.from_pretrained(model_name)
    token_ids = tokenizer.encode(text, add_special_tokens=False)

    chunks: list[str] = []
    start = 0

    while start < len(token_ids):
        end = start + chunk_size
        chunk_ids = token_ids[start:end]
        chunk = tokenizer.decode(chunk_ids, skip_special_tokens=True).strip()

        if chunk:
            chunks.append(chunk)

        start = end - overlap

    return chunks
```

Regla práctica:

```text
Si usas LLMs y embeddings, medir por tokens suele ser más controlable que medir por palabras.
Si usas herramientas simples, medir por caracteres es suficiente como baseline.
```

---

## 13. Segmentación por párrafos

Divide el documento por saltos de línea dobles, conservando unidades naturales.

```python
from __future__ import annotations

import re


def split_paragraphs(text: str) -> list[str]:
    paragraphs = re.split(r"\n\s*\n", text)
    return [paragraph.strip() for paragraph in paragraphs if paragraph.strip()]


def merge_paragraphs(
    paragraphs: list[str],
    max_words: int = 350,
    overlap_paragraphs: int = 1,
) -> list[str]:
    chunks: list[str] = []
    current: list[str] = []
    current_words = 0

    for paragraph in paragraphs:
        paragraph_words = len(paragraph.split())

        if current and current_words + paragraph_words > max_words:
            chunks.append("\n\n".join(current))
            current = current[-overlap_paragraphs:] if overlap_paragraphs else []
            current_words = sum(len(item.split()) for item in current)

        current.append(paragraph)
        current_words += paragraph_words

    if current:
        chunks.append("\n\n".join(current))

    return chunks
```

Uso:

```python
text = """Primer párrafo...

Segundo párrafo...

Tercer párrafo..."""

paragraphs = split_paragraphs(text)
chunks = merge_paragraphs(paragraphs, max_words=300, overlap_paragraphs=1)
```

Ventajas:

```text
Respeta unidades humanas de escritura.
Reduce cortes abruptos.
Funciona bien en documentos narrativos o técnicos simples.
```

Desventajas:

```text
Un párrafo largo puede exceder el límite.
Un párrafo corto puede quedar sin contexto.
No conserva siempre la jerarquía de títulos.
```

---

## 14. Segmentación por secciones

Para documentos con títulos, conviene segmentar por estructura.

### 14.1 Markdown

```python
from __future__ import annotations

import re
from dataclasses import dataclass


@dataclass
class Section:
    heading_path: list[str]
    content: str


def split_markdown_sections(markdown_text: str) -> list[Section]:
    lines = markdown_text.splitlines()
    sections: list[Section] = []
    heading_stack: list[str] = []
    current_lines: list[str] = []

    heading_pattern = re.compile(r"^(#{1,6})\s+(.*)$")

    def flush_current() -> None:
        content = "\n".join(current_lines).strip()
        if content:
            sections.append(
                Section(
                    heading_path=heading_stack.copy(),
                    content=content,
                )
            )

    for line in lines:
        match = heading_pattern.match(line)

        if match:
            flush_current()
            current_lines = []

            level = len(match.group(1))
            title = match.group(2).strip()
            heading_stack = heading_stack[:level - 1]
            heading_stack.append(title)
        else:
            current_lines.append(line)

    flush_current()
    return sections
```

Uso:

```python
from pathlib import Path

markdown_text = Path("manual.md").read_text(encoding="utf-8")
sections = split_markdown_sections(markdown_text)

for section in sections:
    print(section.heading_path)
    print(section.content[:200])
```

### 14.2 Secciones más grandes que el límite

Una sección puede ser demasiado larga. En ese caso:

```text
1. Divide primero por sección.
2. Si la sección cabe, guárdala completa.
3. Si no cabe, divide por párrafos.
4. Si un párrafo no cabe, divide por tokens o caracteres.
5. Conserva heading_path como metadata.
```

```python
from __future__ import annotations


def section_aware_chunking(
    markdown_text: str,
    max_words: int = 350,
) -> list[dict]:
    sections = split_markdown_sections(markdown_text)
    chunks: list[dict] = []

    for section in sections:
        paragraphs = split_paragraphs(section.content)
        section_chunks = merge_paragraphs(paragraphs, max_words=max_words)

        for index, chunk in enumerate(section_chunks):
            chunks.append({
                "heading_path": section.heading_path,
                "section_chunk_index": index,
                "content": chunk,
                "chunk_strategy": "markdown_section_paragraph",
            })

    return chunks
```

---

## 15. Segmentación recursiva

La segmentación recursiva intenta dividir usando separadores en orden de preferencia:

```text
1. Secciones.
2. Párrafos.
3. Líneas.
4. Frases.
5. Palabras.
6. Caracteres.
```

Con LangChain:

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1800,
    chunk_overlap=200,
    separators=["\n## ", "\n### ", "\n\n", "\n", ". ", " ", ""],
    length_function=len,
)

chunks = splitter.split_text(text)
```

La segmentación recursiva es un buen punto de partida porque intenta preservar párrafos y frases antes de cortar por caracteres.

---

## 16. Segmentación semántica con embeddings

La segmentación semántica intenta cortar cuando cambia el tema. Un método simple:

```text
1. Divide el texto en sentencias.
2. Genera embeddings por sentencia.
3. Calcula similitud entre sentencias consecutivas.
4. Corta cuando la similitud cae bajo un umbral.
5. Fusiona segmentos demasiado pequeños.
```

Ejemplo con Sentence Transformers:

```python
from __future__ import annotations

import re
from sentence_transformers import SentenceTransformer
from sklearn.metrics.pairwise import cosine_similarity


def split_sentences(text: str) -> list[str]:
    candidates = re.split(r"(?<=[.!?])\s+", text.strip())
    return [candidate.strip() for candidate in candidates if candidate.strip()]


def semantic_chunking(
    text: str,
    model_name: str = "sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2",
    similarity_threshold: float = 0.55,
    max_sentences_per_chunk: int = 12,
) -> list[str]:
    sentences = split_sentences(text)

    if not sentences:
        return []

    if len(sentences) == 1:
        return sentences

    model = SentenceTransformer(model_name)
    embeddings = model.encode(sentences, normalize_embeddings=True)

    chunks: list[list[str]] = [[sentences[0]]]

    for index in range(1, len(sentences)):
        previous_embedding = embeddings[index - 1].reshape(1, -1)
        current_embedding = embeddings[index].reshape(1, -1)
        similarity = cosine_similarity(previous_embedding, current_embedding)[0][0]

        should_split = (
            similarity < similarity_threshold
            or len(chunks[-1]) >= max_sentences_per_chunk
        )

        if should_split:
            chunks.append([sentences[index]])
        else:
            chunks[-1].append(sentences[index])

    return [" ".join(chunk) for chunk in chunks]
```

Ventajas:

```text
Respeta cambios temáticos.
Puede mejorar precisión en documentos heterogéneos.
Reduce chunks con temas mezclados.
```

Desventajas:

```text
Es más costosa.
Depende del modelo de embeddings.
Requiere ajustar umbrales.
Puede comportarse mal en textos con listas, tablas o código.
```

---

## 17. Segmentación jerárquica

La segmentación jerárquica crea chunks en varios niveles:

```text
Documento
└── Sección grande, 1.500–2.500 tokens
    └── Subsección, 700–1.000 tokens
        └── Chunk final, 250–500 tokens
```

Patrón recomendado:

```text
1. Indexa embeddings en chunks pequeños.
2. Recupera los chunks pequeños más relevantes.
3. Expande contexto hacia el padre: sección, página o bloque superior.
4. Envía al LLM el contexto padre o una mezcla de padre + chunk exacto.
```

Esto mejora:

```text
Precisión de búsqueda.
Contexto suficiente para responder.
Trazabilidad hacia secciones completas.
Reducción de cortes artificiales.
```

### 17.1 Modelo parent-child en PostgreSQL

Ya se soporta con `parent_chunk_id` y `chunk_level`:

```sql
-- Chunk padre: sección completa
INSERT INTO rag_chunks (
    document_id,
    parent_chunk_id,
    chunk_index,
    chunk_level,
    chunk_strategy,
    content,
    content_hash,
    word_count,
    char_count,
    section_title,
    metadata
)
VALUES (
    :document_id,
    NULL,
    1,
    0,
    'section_parent',
    :section_content,
    :section_hash,
    :word_count,
    :char_count,
    :section_title,
    :metadata
)
RETURNING id;
```

```sql
-- Chunk hijo: fragmento pequeño con embedding
INSERT INTO rag_chunks (
    document_id,
    parent_chunk_id,
    chunk_index,
    chunk_level,
    chunk_strategy,
    content,
    content_hash,
    word_count,
    char_count,
    section_title,
    metadata,
    embedding
)
VALUES (
    :document_id,
    :parent_chunk_id,
    :chunk_index,
    1,
    'child_300_words',
    :child_content,
    :child_hash,
    :word_count,
    :char_count,
    :section_title,
    :metadata,
    :embedding
);
```

### 17.2 Recuperar hijo y expandir al padre

```sql
WITH nearest_children AS (
    SELECT
        id,
        parent_chunk_id,
        embedding <=> :query_embedding AS distance
    FROM rag_chunks
    WHERE
        embedding IS NOT NULL
        AND chunk_level = 1
    ORDER BY embedding <=> :query_embedding
    LIMIT 8
)
SELECT
    child.id AS child_id,
    parent.id AS parent_id,
    parent.section_title,
    child.content AS matched_content,
    parent.content AS expanded_context,
    nearest_children.distance
FROM nearest_children
JOIN rag_chunks child ON child.id = nearest_children.id
LEFT JOIN rag_chunks parent ON parent.id = child.parent_chunk_id
ORDER BY nearest_children.distance;
```

### 17.3 Ejemplo con LlamaIndex

```python
from llama_index.core import Document
from llama_index.core.node_parser import HierarchicalNodeParser

text = "Contenido del documento..."
documents = [Document(text=text, metadata={"source": "manual.md"})]

parser = HierarchicalNodeParser.from_defaults(
    chunk_sizes=[2048, 512, 128],
    chunk_overlap=20,
)

nodes = parser.get_nodes_from_documents(documents)

for node in nodes:
    print(node.node_id)
    print(node.metadata)
    print(node.text[:120])
```

Uso recomendado:

```text
Documentos técnicos largos.
Normativas.
Contratos.
Manuales.
Papers.
Documentos donde la respuesta necesita contexto mayor que el chunk exacto.
```

---

## 18. Modelos de embeddings y herramientas de IA

### 18.1 BERT

BERT clásico es un encoder de lenguaje. Puede generar representaciones vectoriales, pero no todos los modelos BERT están optimizados para búsqueda semántica. Para RAG, suele ser mejor usar modelos entrenados explícitamente para embeddings de oraciones o recuperación.

Ejemplo de embedding con BERT usando mean pooling:

```python
from __future__ import annotations

import torch
from transformers import AutoModel, AutoTokenizer


def mean_pooling(model_output, attention_mask):
    token_embeddings = model_output.last_hidden_state
    input_mask_expanded = attention_mask.unsqueeze(-1).expand(token_embeddings.size()).float()
    return torch.sum(token_embeddings * input_mask_expanded, 1) / torch.clamp(
        input_mask_expanded.sum(1),
        min=1e-9,
    )


def embed_with_bert(
    texts: list[str],
    model_name: str = "bert-base-multilingual-cased",
) -> list[list[float]]:
    tokenizer = AutoTokenizer.from_pretrained(model_name)
    model = AutoModel.from_pretrained(model_name)
    model.eval()

    encoded_input = tokenizer(
        texts,
        padding=True,
        truncation=True,
        return_tensors="pt",
        max_length=512,
    )

    with torch.no_grad():
        model_output = model(**encoded_input)

    sentence_embeddings = mean_pooling(model_output, encoded_input["attention_mask"])
    sentence_embeddings = torch.nn.functional.normalize(sentence_embeddings, p=2, dim=1)

    return sentence_embeddings.tolist()
```

Advertencia:

```text
Este método sirve como ejemplo educativo.
Para producción, valida contra un modelo de sentence embeddings.
```

### 18.2 BETO para español

BETO es un BERT entrenado en corpus de español. Puede ser útil para tareas de NLP en español, clasificación, extracción o representación inicial. Para búsqueda semántica pura, también conviene evaluar modelos multilingües o españoles ajustados para embeddings.

Ejemplo con BETO:

```python
embeddings = embed_with_bert(
    texts=[
        "PostgreSQL permite búsqueda vectorial con pgvector.",
        "La segmentación jerárquica conserva contexto documental.",
    ],
    model_name="dccuchile/bert-base-spanish-wwm-cased",
)

print(len(embeddings[0]))  # normalmente 768
```

Uso recomendado de BETO:

```text
Clasificación de textos en español.
Extracción de entidades.
Segmentación semántica experimental.
Baseline local para embeddings en español.
Fine-tuning propio si tienes datos etiquetados.
```

Uso no ideal:

```text
Usarlo sin evaluación como único motor de embeddings para RAG productivo.
```

### 18.3 Sentence Transformers

Para RAG, una alternativa práctica es usar `sentence-transformers` con modelos diseñados para embeddings.

Ejemplo:

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2")

texts = [
    "PostgreSQL puede almacenar embeddings con pgvector.",
    "RAG combina recuperación documental con generación de respuestas.",
]

embeddings = model.encode(
    texts,
    normalize_embeddings=True,
).tolist()
```

Modelos a evaluar:

```text
sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2
intfloat/multilingual-e5-base
intfloat/multilingual-e5-large
BAAI/bge-m3
jinaai/jina-embeddings-v3
```

Notas:

```text
E5 suele funcionar bien con prefijos como "query:" y "passage:".
BGE-M3 es útil para escenarios multilingües y recuperación densa/sparse/multivector.
Modelos comerciales pueden ser útiles si necesitas menos operación local.
La dimensión del modelo debe coincidir con vector(n) en PostgreSQL.
```

### 18.4 Ejemplo con prefijos tipo E5

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("intfloat/multilingual-e5-base")

documents = [
    "passage: PostgreSQL soporta JSONB, full-text search y extensiones como pgvector.",
    "passage: La segmentación por secciones mejora la trazabilidad de respuestas RAG.",
]

query = "query: ¿Cómo puedo hacer búsqueda vectorial en PostgreSQL?"

document_embeddings = model.encode(documents, normalize_embeddings=True)
query_embedding = model.encode([query], normalize_embeddings=True)[0]
```

---

## 19. Inserción de embeddings en PostgreSQL desde Python

Dependencias:

```bash
pip install psycopg[binary] pgvector sentence-transformers python-dotenv
```

Ejemplo:

```python
from __future__ import annotations

import hashlib
import os
from dataclasses import dataclass

import psycopg
from dotenv import load_dotenv
from pgvector.psycopg import register_vector
from sentence_transformers import SentenceTransformer

load_dotenv()

DATABASE_URL = os.environ["DATABASE_URL"]
EMBEDDING_MODEL_NAME = "intfloat/multilingual-e5-base"


@dataclass
class ChunkInput:
    document_id: str
    chunk_index: int
    content: str
    section_title: str | None = None
    metadata: dict | None = None


def sha256_text(text: str) -> str:
    return hashlib.sha256(text.encode("utf-8")).hexdigest()


def insert_chunks(chunks: list[ChunkInput]) -> None:
    model = SentenceTransformer(EMBEDDING_MODEL_NAME)
    texts = [f"passage: {chunk.content}" for chunk in chunks]
    embeddings = model.encode(texts, normalize_embeddings=True).tolist()

    with psycopg.connect(DATABASE_URL) as conn:
        register_vector(conn)

        with conn.cursor() as cur:
            for chunk, embedding in zip(chunks, embeddings, strict=True):
                content_hash = sha256_text(chunk.content)
                word_count = len(chunk.content.split())
                char_count = len(chunk.content)

                cur.execute(
                    """
                    INSERT INTO rag_chunks (
                        document_id,
                        chunk_index,
                        chunk_level,
                        chunk_strategy,
                        content,
                        content_hash,
                        word_count,
                        char_count,
                        section_title,
                        metadata,
                        embedding
                    )
                    VALUES (
                        %(document_id)s,
                        %(chunk_index)s,
                        0,
                        'fixed_or_section_chunking',
                        %(content)s,
                        %(content_hash)s,
                        %(word_count)s,
                        %(char_count)s,
                        %(section_title)s,
                        %(metadata)s,
                        %(embedding)s
                    )
                    ON CONFLICT (document_id, chunk_index, chunk_level, content_hash)
                    DO NOTHING
                    """,
                    {
                        "document_id": chunk.document_id,
                        "chunk_index": chunk.chunk_index,
                        "content": chunk.content,
                        "content_hash": content_hash,
                        "word_count": word_count,
                        "char_count": char_count,
                        "section_title": chunk.section_title,
                        "metadata": chunk.metadata or {},
                        "embedding": embedding,
                    },
                )

        conn.commit()
```

---

## 20. Consulta RAG desde Python

```python
from __future__ import annotations

import os

import psycopg
from dotenv import load_dotenv
from pgvector.psycopg import register_vector
from sentence_transformers import SentenceTransformer

load_dotenv()

DATABASE_URL = os.environ["DATABASE_URL"]
EMBEDDING_MODEL_NAME = "intfloat/multilingual-e5-base"


def retrieve_context(
    query: str,
    limit: int = 8,
    department: str | None = None,
) -> list[dict]:
    model = SentenceTransformer(EMBEDDING_MODEL_NAME)
    query_embedding = model.encode(
        [f"query: {query}"],
        normalize_embeddings=True,
    )[0].tolist()

    filters = [
        "d.deleted_at IS NULL",
        "c.embedding IS NOT NULL",
    ]

    params = {
        "query_embedding": query_embedding,
        "limit": limit,
    }

    if department:
        filters.append("d.metadata @> %(department_filter)s::jsonb")
        params["department_filter"] = {"department": department}

    sql = f"""
        SELECT
            c.id,
            d.title,
            c.section_title,
            c.content,
            c.metadata,
            c.embedding <=> %(query_embedding)s AS distance
        FROM rag_chunks c
        JOIN rag_documents d ON d.id = c.document_id
        WHERE {' AND '.join(filters)}
        ORDER BY c.embedding <=> %(query_embedding)s
        LIMIT %(limit)s
    """

    with psycopg.connect(DATABASE_URL) as conn:
        register_vector(conn)

        with conn.cursor(row_factory=psycopg.rows.dict_row) as cur:
            cur.execute(sql, params)
            return list(cur.fetchall())
```

Construcción de prompt:

```python

def build_rag_prompt(query: str, contexts: list[dict]) -> str:
    context_text = "\n\n".join(
        f"[Fuente {index + 1}] {item['title']} / {item['section_title']}\n{item['content']}"
        for index, item in enumerate(contexts)
    )

    return f"""
Responde usando solo el contexto entregado.
Si el contexto no contiene la respuesta, indica que no hay información suficiente.
Cita las fuentes por número cuando corresponda.

Contexto:
{context_text}

Pregunta:
{query}
""".strip()
```

---

## 21. Procesamiento de documentos por tipo

### 21.1 TXT

```python
from pathlib import Path


def read_txt(path: str) -> str:
    return Path(path).read_text(encoding="utf-8")
```

### 21.2 Markdown

```python
from pathlib import Path


def read_markdown(path: str) -> str:
    return Path(path).read_text(encoding="utf-8")
```

Para Markdown, conserva títulos como metadatos:

```python
markdown_text = read_markdown("guia.md")
chunks = section_aware_chunking(markdown_text, max_words=350)
```

### 21.3 DOCX

Dependencia:

```bash
pip install python-docx
```

Ejemplo:

```python
from docx import Document


def read_docx(path: str) -> str:
    document = Document(path)
    paragraphs = [paragraph.text.strip() for paragraph in document.paragraphs]
    return "\n\n".join(paragraph for paragraph in paragraphs if paragraph)
```

Para `.doc` antiguo:

```text
Convierte primero a .docx, .pdf o .txt.
Puedes usar LibreOffice en modo headless, antiword o una etapa externa controlada.
No trates .doc como si fuera .docx.
```

Ejemplo con LibreOffice:

```bash
libreoffice --headless --convert-to docx archivo.doc --outdir ./convertidos
```

### 21.4 PDF con PyMuPDF

Dependencia:

```bash
pip install pymupdf
```

Ejemplo:

```python
import pymupdf


def read_pdf_with_pymupdf(path: str) -> list[dict]:
    document = pymupdf.open(path)
    pages: list[dict] = []

    for page_index, page in enumerate(document, start=1):
        text = page.get_text("text").strip()

        if text:
            pages.append({
                "page_number": page_index,
                "text": text,
            })

    return pages
```

Uso:

```python
pages = read_pdf_with_pymupdf("manual.pdf")

for page in pages:
    paragraphs = split_paragraphs(page["text"])
    chunks = merge_paragraphs(paragraphs, max_words=300)
```

### 21.5 PDF con pypdf

Dependencia:

```bash
pip install pypdf
```

Ejemplo:

```python
from pypdf import PdfReader


def read_pdf_with_pypdf(path: str) -> list[dict]:
    reader = PdfReader(path)
    pages: list[dict] = []

    for page_index, page in enumerate(reader.pages, start=1):
        text = page.extract_text() or ""
        text = text.strip()

        if text:
            pages.append({
                "page_number": page_index,
                "text": text,
            })

    return pages
```

### 21.6 PDFs escaneados

Para PDFs escaneados necesitas OCR.

Opciones:

```text
Tesseract + pytesseract.
OCRmyPDF.
Unstructured con estrategia OCR.
Servicios cloud de OCR si el volumen o calidad lo justifica.
Modelos de visión si el documento tiene layout complejo.
```

Regla práctica:

```text
Primero detecta si el PDF tiene texto embebido.
Si no tiene texto, aplica OCR.
Guarda metadata indicando ocr=true.
Evalúa calidad de OCR antes de indexar masivamente.
```

### 21.7 Unstructured para múltiples formatos

Dependencia:

```bash
pip install unstructured
```

Ejemplo:

```python
from unstructured.partition.auto import partition


def read_with_unstructured(path: str) -> list[dict]:
    elements = partition(filename=path)
    records: list[dict] = []

    for index, element in enumerate(elements):
        text = str(element).strip()

        if not text:
            continue

        records.append({
            "element_index": index,
            "element_type": element.category,
            "text": text,
            "metadata": element.metadata.to_dict(),
        })

    return records
```

Este enfoque es útil cuando quieres una sola interfaz para:

```text
txt, md, html, pdf, doc, docx, ppt, pptx, xlsx, email, epub, xml, rtf y otros.
```

### 21.8 HTML

```python
from bs4 import BeautifulSoup


def read_html(path: str) -> str:
    with open(path, "r", encoding="utf-8") as file:
        soup = BeautifulSoup(file, "html.parser")

    for element in soup(["script", "style", "nav", "footer"]):
        element.decompose()

    return soup.get_text(separator="\n", strip=True)
```

Dependencia:

```bash
pip install beautifulsoup4
```

### 21.9 CSV

Para CSV, no siempre conviene convertir toda la tabla a texto plano. A veces conviene crear chunks por fila, por grupo o por resumen.

```python
import pandas as pd


def read_csv_as_row_documents(path: str) -> list[dict]:
    df = pd.read_csv(path)
    records: list[dict] = []

    for index, row in df.iterrows():
        content = "\n".join(f"{column}: {row[column]}" for column in df.columns)
        records.append({
            "row_index": int(index),
            "content": content,
            "metadata": {
                "columns": list(df.columns),
            },
        })

    return records
```

Uso recomendado:

```text
Catálogos: chunk por fila.
Tablas largas: chunk por grupo lógico.
Tablas numéricas: almacena datos estructurados y genera texto descriptivo solo para búsqueda.
```

### 21.10 XLSX

```python
import pandas as pd


def read_xlsx_sheets(path: str) -> list[dict]:
    sheets = pd.read_excel(path, sheet_name=None)
    records: list[dict] = []

    for sheet_name, df in sheets.items():
        for index, row in df.iterrows():
            content = "\n".join(f"{column}: {row[column]}" for column in df.columns)
            records.append({
                "sheet_name": sheet_name,
                "row_index": int(index),
                "content": content,
            })

    return records
```

### 21.11 PowerPoint

Para `pptx`, puedes usar `python-pptx` o `unstructured`.

```bash
pip install python-pptx
```

```python
from pptx import Presentation


def read_pptx(path: str) -> list[dict]:
    presentation = Presentation(path)
    slides: list[dict] = []

    for slide_index, slide in enumerate(presentation.slides, start=1):
        texts: list[str] = []

        for shape in slide.shapes:
            if hasattr(shape, "text"):
                text = shape.text.strip()
                if text:
                    texts.append(text)

        if texts:
            slides.append({
                "slide_number": slide_index,
                "text": "\n".join(texts),
            })

    return slides
```

---

## 22. Pipeline completo de ingesta RAG

Flujo recomendado:

```text
1. Detectar archivo.
2. Calcular checksum.
3. Registrar documento.
4. Extraer texto.
5. Normalizar texto.
6. Detectar idioma.
7. Segmentar.
8. Enriquecer metadatos.
9. Generar embeddings.
10. Insertar chunks.
11. Crear o refrescar índices.
12. Ejecutar pruebas de recuperación.
```

Ejemplo conceptual:

```python
from __future__ import annotations

import hashlib
from pathlib import Path


def file_checksum(path: str) -> str:
    data = Path(path).read_bytes()
    return hashlib.sha256(data).hexdigest()


def normalize_text(text: str) -> str:
    text = text.replace("\r\n", "\n").replace("\r", "\n")
    text = "\n".join(line.rstrip() for line in text.splitlines())
    return text.strip()


def process_document(path: str) -> list[dict]:
    suffix = Path(path).suffix.lower()

    if suffix == ".txt":
        text = read_txt(path)
        chunks = chunk_by_words(normalize_text(text), chunk_size=300, overlap=50)
        return [
            {
                "content": chunk,
                "metadata": {"source_type": "txt"},
            }
            for chunk in chunks
        ]

    if suffix == ".md":
        text = read_markdown(path)
        return section_aware_chunking(normalize_text(text), max_words=350)

    if suffix == ".docx":
        text = read_docx(path)
        chunks = chunk_by_words(normalize_text(text), chunk_size=300, overlap=50)
        return [
            {
                "content": chunk,
                "metadata": {"source_type": "docx"},
            }
            for chunk in chunks
        ]

    if suffix == ".pdf":
        pages = read_pdf_with_pymupdf(path)
        records: list[dict] = []

        for page in pages:
            page_text = normalize_text(page["text"])
            page_chunks = chunk_by_words(page_text, chunk_size=300, overlap=50)

            for chunk in page_chunks:
                records.append({
                    "content": chunk,
                    "metadata": {
                        "source_type": "pdf",
                        "page_number": page["page_number"],
                    },
                })

        return records

    records = read_with_unstructured(path)
    return [
        {
            "content": record["text"],
            "metadata": {
                "source_type": suffix.lstrip(".") or "other",
                "element_type": record["element_type"],
                **record["metadata"],
            },
        }
        for record in records
    ]
```

---

## 23. Re-ranking

Después de recuperar 20–100 candidatos, puedes reordenarlos con un modelo más caro pero más preciso.

Flujo:

```text
1. PostgreSQL recupera candidatos rápidos.
2. Cross-encoder evalúa pares pregunta/chunk.
3. Se ordenan por score del cross-encoder.
4. Se envían los mejores al LLM.
```

Ejemplo:

```python
from sentence_transformers import CrossEncoder


def rerank(query: str, candidates: list[dict], top_n: int = 8) -> list[dict]:
    reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")
    pairs = [(query, candidate["content"]) for candidate in candidates]
    scores = reranker.predict(pairs)

    scored_candidates = [
        {
            **candidate,
            "rerank_score": float(score),
        }
        for candidate, score in zip(candidates, scores, strict=True)
    ]

    return sorted(
        scored_candidates,
        key=lambda item: item["rerank_score"],
        reverse=True,
    )[:top_n]
```

Para español, evalúa modelos multilingües o entrena un reranker propio si tienes datos de relevancia.

---

## 24. Buenas prácticas de permisos

En RAG, el retrieval es una superficie de seguridad. Si un usuario no puede ver un documento, ese documento no debe recuperarse.

Filtra por:

```text
tenant_id
organization_id
project_id
user_id
role
classification
language
document_status
deleted_at
valid_from / valid_to
```

Ejemplo:

```sql
SELECT
    c.id,
    c.content
FROM rag_chunks c
JOIN rag_documents d ON d.id = c.document_id
WHERE
    d.deleted_at IS NULL
    AND d.metadata ->> 'tenant_id' = :tenant_id
    AND c.metadata -> 'allowed_roles' ? :user_role
ORDER BY c.embedding <=> :query_embedding
LIMIT 8;
```

Regla crítica:

```text
La autorización debe aplicarse antes o durante la búsqueda.
No basta con pedirle al LLM que ignore información sensible.
```

---

## 25. Calidad de datos para RAG

Antes de indexar, valida:

```text
[ ] El texto no está vacío.
[ ] El documento tiene idioma detectado.
[ ] El documento tiene título o identificador trazable.
[ ] El checksum está calculado.
[ ] El contenido duplicado fue detectado.
[ ] Las páginas están preservadas si vienen de PDF.
[ ] Las tablas importantes no se perdieron.
[ ] Las secciones tienen heading_path.
[ ] El OCR tiene calidad suficiente.
[ ] Los metadatos de permisos están completos.
```

Tabla de calidad opcional:

```sql
CREATE TABLE rag_document_quality_checks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id UUID NOT NULL REFERENCES rag_documents(id) ON DELETE CASCADE,
    check_name TEXT NOT NULL,
    check_status TEXT NOT NULL CHECK (check_status IN ('pass', 'warn', 'fail')),
    details JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## 26. Evaluación de retrieval

No evalúes RAG solo leyendo respuestas finales. Evalúa retrieval por separado.

Crea un set de preguntas con chunks esperados:

```sql
CREATE TABLE rag_eval_questions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    question TEXT NOT NULL,
    expected_chunk_ids UUID[] NOT NULL,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Métricas útiles:

```text
Recall@k      # si el chunk correcto aparece dentro de los k recuperados
Precision@k   # proporción de resultados relevantes
MRR           # qué tan arriba aparece el primer resultado relevante
nDCG          # ranking ponderado por relevancia
latency p50/p95/p99
coverage      # documentos o dominios sin recuperación
faithfulness  # si la respuesta se sostiene en el contexto
```

Checklist de evaluación:

```text
[ ] Evalúa vector search sola.
[ ] Evalúa full-text search sola.
[ ] Evalúa búsqueda híbrida.
[ ] Evalúa con y sin reranker.
[ ] Evalúa diferentes tamaños de chunk.
[ ] Evalúa diferentes overlaps.
[ ] Evalúa diferentes modelos de embedding.
[ ] Evalúa por idioma y por tipo documental.
```

---

## 27. Versionado de embeddings

Cuando cambias el modelo de embeddings, no mezcles vectores incompatibles en la misma columna sin trazabilidad.

Opción simple:

```sql
ALTER TABLE rag_chunks
ADD COLUMN embedding_model_name TEXT;

ALTER TABLE rag_chunks
ADD COLUMN embedding_created_at TIMESTAMPTZ;
```

Opción más limpia:

```sql
CREATE TABLE rag_chunk_embeddings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    chunk_id UUID NOT NULL REFERENCES rag_chunks(id) ON DELETE CASCADE,
    embedding_model_name TEXT NOT NULL,
    embedding_dimension INTEGER NOT NULL,
    embedding vector(768) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (chunk_id, embedding_model_name)
);
```

Nota:

```text
PostgreSQL necesita dimensión fija por columna vector(n).
Si usas modelos con dimensiones distintas, considera tablas separadas o columnas separadas.
```

---

## 28. Actualización y borrado de documentos

Para documentos cambiantes:

```text
1. Calcula checksum del archivo nuevo.
2. Si checksum ya existe, no reproceses.
3. Si cambió, marca versión anterior como reemplazada o deleted_at.
4. Inserta nueva versión.
5. Reprocesa chunks.
6. Registra lineage entre versiones.
```

Ejemplo:

```sql
ALTER TABLE rag_documents
ADD COLUMN previous_document_id UUID REFERENCES rag_documents(id);

ALTER TABLE rag_documents
ADD COLUMN document_status TEXT NOT NULL DEFAULT 'active'
CHECK (document_status IN ('active', 'replaced', 'deleted', 'draft'));
```

Borrado lógico:

```sql
UPDATE rag_documents
SET
    deleted_at = now(),
    document_status = 'deleted'
WHERE id = :document_id;
```

Regla práctica:

```text
No borres físicamente documentos indexados si necesitas auditoría.
Usa borrado físico solo cuando políticas de privacidad o retención lo exijan.
```

---

## 29. Observabilidad específica para RAG

Mide como mínimo:

```text
Ingesta:
[ ] documentos procesados por hora
[ ] errores de parsing por tipo documental
[ ] documentos con OCR
[ ] tasa de duplicados
[ ] chunks generados por documento
[ ] embeddings pendientes

Retrieval:
[ ] latencia de embedding de query
[ ] latencia SQL
[ ] top-k promedio
[ ] filtros aplicados
[ ] número de resultados vacíos
[ ] distancia promedio de resultados

Generación:
[ ] tokens de contexto
[ ] tokens de salida
[ ] costo por consulta
[ ] respuestas sin evidencia
[ ] feedback negativo
```

Consulta para chunks sin embedding:

```sql
SELECT
    d.source_type,
    count(*) AS pending_chunks
FROM rag_chunks c
JOIN rag_documents d ON d.id = c.document_id
WHERE c.embedding IS NULL
GROUP BY d.source_type
ORDER BY pending_chunks DESC;
```

Consulta para documentos con muchos chunks:

```sql
SELECT
    d.id,
    d.title,
    count(c.id) AS chunk_count
FROM rag_documents d
JOIN rag_chunks c ON c.document_id = d.id
GROUP BY d.id, d.title
ORDER BY chunk_count DESC
LIMIT 20;
```

---

## 30. Rendimiento y operación

Buenas prácticas:

```text
[ ] Carga datos primero y crea índices vectoriales después cuando sea posible.
[ ] Usa CREATE INDEX CONCURRENTLY en producción.
[ ] Ajusta hnsw.ef_search o ivfflat.probes con evaluación, no por intuición.
[ ] Usa filtros relacionales selectivos antes del retrieval cuando aplique.
[ ] Considera particionar por tenant, dominio o fecha si el volumen crece.
[ ] Evita chunks excesivamente pequeños: aumentan costo y ruido.
[ ] Evita chunks excesivamente grandes: reducen precisión.
[ ] Mantén ANALYZE actualizado tras cargas masivas.
[ ] Evalúa EXPLAIN ANALYZE en consultas críticas.
```

Crear índices sin bloquear escrituras:

```sql
CREATE INDEX CONCURRENTLY idx_rag_chunks_embedding_hnsw_cosine_prod
ON rag_chunks
USING hnsw (embedding vector_cosine_ops)
WHERE embedding IS NOT NULL;
```

Actualizar estadísticas:

```sql
ANALYZE rag_documents;
ANALYZE rag_chunks;
```

Ver plan:

```sql
EXPLAIN ANALYZE
SELECT id, content
FROM rag_chunks
WHERE embedding IS NOT NULL
ORDER BY embedding <=> :query_embedding
LIMIT 10;
```

---

## 31. Anti-patrones frecuentes

```text
[ ] Indexar documentos sin limpiar texto.
[ ] Usar un solo chunk por documento largo.
[ ] Usar chunks diminutos sin contexto.
[ ] No guardar source_uri, página o sección.
[ ] No guardar modelo ni dimensión del embedding.
[ ] Mezclar embeddings de modelos distintos sin control.
[ ] Usar JSONB para campos que deberían ser columnas.
[ ] No filtrar por permisos antes de recuperar contexto.
[ ] No evaluar retrieval con preguntas reales.
[ ] Confiar solo en búsqueda vectorial e ignorar búsqueda textual.
[ ] Reprocesar documentos sin checksum ni control de idempotencia.
[ ] No tener estrategia para documentos reemplazados o eliminados.
```

---

## 32. Estrategia recomendada por madurez

### Nivel 1: prototipo

```text
TXT/Markdown/PDF simple.
Chunking por 300 palabras con overlap 50.
Embeddings con sentence-transformers multilingüe.
PostgreSQL + pgvector.
Búsqueda vectorial top-k.
Metadatos básicos en JSONB.
```

### Nivel 2: producto interno

```text
Chunking por secciones y párrafos.
Búsqueda híbrida vector + full-text.
Filtros por permisos.
Logs de retrieval.
Evaluación con golden set.
Re-ranking para preguntas críticas.
Control de versiones por checksum.
```

### Nivel 3: producción exigente

```text
Chunking jerárquico.
Reranker especializado.
Particionado por tenant o dominio.
Evaluación continua.
Observabilidad p95/p99.
Políticas de retención y privacidad.
Auditoría de fuentes usadas por respuesta.
Pipeline incremental e idempotente.
```

---

## 33. Tutorial mínimo de extremo a extremo

### 33.1 Crear base

```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE EXTENSION IF NOT EXISTS pgcrypto;

CREATE TABLE rag_documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_uri TEXT NOT NULL,
    source_type TEXT NOT NULL,
    title TEXT,
    language TEXT NOT NULL DEFAULT 'es',
    checksum TEXT NOT NULL,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    ingested_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at TIMESTAMPTZ,
    UNIQUE (source_uri, checksum)
);

CREATE TABLE rag_chunks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id UUID NOT NULL REFERENCES rag_documents(id) ON DELETE CASCADE,
    chunk_index INTEGER NOT NULL,
    content TEXT NOT NULL,
    content_hash TEXT NOT NULL,
    word_count INTEGER NOT NULL,
    char_count INTEGER NOT NULL,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    embedding vector(768),
    search_vector tsvector GENERATED ALWAYS AS (
        to_tsvector('spanish', coalesce(content, ''))
    ) STORED,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (document_id, chunk_index, content_hash)
);

CREATE INDEX idx_rag_chunks_search_vector
ON rag_chunks
USING GIN (search_vector);

CREATE INDEX idx_rag_chunks_metadata_gin
ON rag_chunks
USING GIN (metadata jsonb_path_ops);

CREATE INDEX idx_rag_chunks_embedding_hnsw_cosine
ON rag_chunks
USING hnsw (embedding vector_cosine_ops)
WHERE embedding IS NOT NULL;
```

### 33.2 Instalar dependencias Python

```bash
pip install psycopg[binary] pgvector sentence-transformers pymupdf python-dotenv
```

### 33.3 Ingestar un Markdown simple

```python
from __future__ import annotations

import hashlib
import os
from pathlib import Path

import psycopg
from dotenv import load_dotenv
from pgvector.psycopg import register_vector
from sentence_transformers import SentenceTransformer

load_dotenv()
DATABASE_URL = os.environ["DATABASE_URL"]
MODEL_NAME = "intfloat/multilingual-e5-base"


def sha256_text(text: str) -> str:
    return hashlib.sha256(text.encode("utf-8")).hexdigest()


def chunk_by_words(text: str, chunk_size: int = 300, overlap: int = 50) -> list[str]:
    words = text.split()
    chunks = []
    start = 0

    while start < len(words):
        end = start + chunk_size
        chunks.append(" ".join(words[start:end]))
        start = end - overlap

    return [chunk for chunk in chunks if chunk.strip()]


def ingest_markdown(path: str) -> None:
    model = SentenceTransformer(MODEL_NAME)
    text = Path(path).read_text(encoding="utf-8")
    checksum = sha256_text(text)
    chunks = chunk_by_words(text)
    embeddings = model.encode(
        [f"passage: {chunk}" for chunk in chunks],
        normalize_embeddings=True,
    ).tolist()

    with psycopg.connect(DATABASE_URL) as conn:
        register_vector(conn)

        with conn.cursor() as cur:
            cur.execute(
                """
                INSERT INTO rag_documents (
                    source_uri,
                    source_type,
                    title,
                    checksum,
                    metadata
                )
                VALUES (%s, 'markdown', %s, %s, %s)
                ON CONFLICT (source_uri, checksum)
                DO UPDATE SET ingested_at = now()
                RETURNING id
                """,
                (path, Path(path).stem, checksum, {"embedding_model": MODEL_NAME}),
            )
            document_id = cur.fetchone()[0]

            for index, (chunk, embedding) in enumerate(zip(chunks, embeddings, strict=True)):
                cur.execute(
                    """
                    INSERT INTO rag_chunks (
                        document_id,
                        chunk_index,
                        content,
                        content_hash,
                        word_count,
                        char_count,
                        metadata,
                        embedding
                    )
                    VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
                    ON CONFLICT (document_id, chunk_index, content_hash)
                    DO NOTHING
                    """,
                    (
                        document_id,
                        index,
                        chunk,
                        sha256_text(chunk),
                        len(chunk.split()),
                        len(chunk),
                        {"source_type": "markdown"},
                        embedding,
                    ),
                )

        conn.commit()


if __name__ == "__main__":
    ingest_markdown("guia.md")
```

### 33.4 Consultar

```python
from __future__ import annotations

import os

import psycopg
from dotenv import load_dotenv
from pgvector.psycopg import register_vector
from sentence_transformers import SentenceTransformer

load_dotenv()
DATABASE_URL = os.environ["DATABASE_URL"]
MODEL_NAME = "intfloat/multilingual-e5-base"


def search(query: str, limit: int = 5) -> list[dict]:
    model = SentenceTransformer(MODEL_NAME)
    query_embedding = model.encode([f"query: {query}"], normalize_embeddings=True)[0].tolist()

    with psycopg.connect(DATABASE_URL) as conn:
        register_vector(conn)

        with conn.cursor(row_factory=psycopg.rows.dict_row) as cur:
            cur.execute(
                """
                SELECT
                    d.title,
                    c.content,
                    c.embedding <=> %s AS distance
                FROM rag_chunks c
                JOIN rag_documents d ON d.id = c.document_id
                WHERE
                    d.deleted_at IS NULL
                    AND c.embedding IS NOT NULL
                ORDER BY c.embedding <=> %s
                LIMIT %s
                """,
                (query_embedding, query_embedding, limit),
            )
            return list(cur.fetchall())


if __name__ == "__main__":
    results = search("¿Cómo se configuran índices vectoriales en PostgreSQL?")

    for result in results:
        print(result["title"], result["distance"])
        print(result["content"][:500])
        print("---")
```

---

## 34. Checklist final

Antes de dar por listo un RAG con PostgreSQL:

```text
[ ] Los documentos tienen checksum.
[ ] Los documentos tienen source_uri trazable.
[ ] Los chunks tienen página, sección o heading_path cuando aplica.
[ ] El modelo de embedding está registrado.
[ ] La dimensión vector(n) coincide con el modelo.
[ ] Existe índice vectorial para la métrica usada.
[ ] Existe índice full-text si hay búsqueda híbrida.
[ ] JSONB tiene índices para filtros frecuentes.
[ ] Los permisos se aplican en SQL.
[ ] Hay evaluación de retrieval con preguntas reales.
[ ] Hay logs de consultas y chunks recuperados.
[ ] Hay estrategia de reingesta y versionado.
[ ] Hay monitoreo de latencia y resultados vacíos.
[ ] Hay política para PII y documentos sensibles.
```

---

## 35. Fuentes técnicas consultadas

- PostgreSQL Documentation: JSON Types, JSONB indexing, GIN indexes y full-text search.
- PostgreSQL Documentation: Preferred Index Types for Text Search.
- pgvector README: HNSW, IVFFlat, operadores de distancia, filtros, búsqueda híbrida y recomendaciones de indexado.
- LangChain documentation: RecursiveCharacterTextSplitter.
- LlamaIndex documentation: HierarchicalNodeParser.
- Hugging Face model card: BETO, `dccuchile/bert-base-spanish-wwm-cased`.
- Sentence Transformers documentation: semantic search y embeddings.
- PyMuPDF documentation: extracción de texto desde PDF.
- pypdf documentation: lectura y extracción de texto desde PDF.
- python-docx documentation: lectura de documentos DOCX.
- Unstructured documentation: partitioning de documentos por tipo.
