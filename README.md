# Tutos

Repositorio personal de tutoriales técnicos, guías prácticas y apuntes de referencia para desarrollo de software, datos, frontend, backend, infraestructura, seguridad, QA, agentes de IA, administración de servidores y gestión de proyectos.

El objetivo de este repositorio es centralizar material de estudio y consulta rápida en español, con documentos orientados a aplicación práctica. Las guías combinan fundamentos, ejemplos de código, comandos, configuraciones, checklists, criterios de operación y recomendaciones para proyectos reales.

---

## Índice rápido

| Categoría | Contenido |
|---|---|
| [Python, calidad de código y diseño](#python-calidad-de-código-y-diseño) | PEP 8, QA/testing, patrones de diseño y debugging. |
| [Backend, APIs y bases de datos](#backend-apis-y-bases-de-datos) | FastAPI, PostgreSQL, instalación en Ubuntu, datos, RAG y ciencia de datos. |
| [Frontend, web design y SEO](#frontend-web-design-y-seo) | CSS, React, Next.js y SEO/campañas digitales. |
| [Seguridad, infraestructura y operaciones](#seguridad-infraestructura-y-operaciones) | Ciberseguridad, AWS, servicios Ubuntu, Apache, SSH, agentes IA y operación. |
| [Git, gestión y procesos](#git-gestión-y-procesos) | Git, gestión de proyectos, producto, Scrum, marketing y metodologías ágiles. |
| [Rutas de aprendizaje](#rutas-de-aprendizaje-sugeridas) | Secuencias recomendadas para estudiar por objetivo. |

---

## Estructura del repositorio

```text
Tutos/
├── Backend-APIs-DB/
├── Frontend-WebDesign/
├── Git-Management-Processes/
├── Python-CodeQuiality-Design/
├── Security-Infrastructure-Operations/
└── README.md
```

> Nota: la carpeta `Python-CodeQuiality-Design` conserva el nombre actual del repositorio. Si se renombra en el futuro a `Python-CodeQuality-Design`, habrá que actualizar los enlaces de este README.

---

## Python, calidad de código y diseño

| Guía | Tema |
|---|---|
| [Guía práctica de PEP 8 para Python](Python-CodeQuiality-Design/guia_pep8_python.md) | Estilo Python, naming, imports, docstrings, type hints, linters, formatters y configuración con Ruff, Black, isort, flake8 y pycodestyle. |
| [Guía de QA y testing de sistemas con Python](Python-CodeQuiality-Design/guia_qa_testing_python.md) | Unit tests, integration tests, smoke tests, debugging, logging, coverage, CI/CD, monitoreo y testing avanzado. |
| [Guía de los 23 patrones de diseño](Python-CodeQuiality-Design/guia_23_patrones_diseno_python_c_cpp_csharp.md) | Patrones GoF creacionales, estructurales y de comportamiento, con ejemplos en Python, C, C++ y C#. |
| [Comandos básicos de `breakpoint()` en Python](Python-CodeQuiality-Design/breakpointPython.md) | Apunte breve sobre depuración interactiva con `breakpoint()` en Python. |

---

## Backend, APIs y bases de datos

| Guía | Tema |
|---|---|
| [Guía de buenas prácticas para APIs con FastAPI](Backend-APIs-DB/guia_buenas_practicas_fastapi_completa.md) | FastAPI, validaciones, modelos, errores, códigos de estado, dependencias, OAuth2, SQLModel, middlewares, templates y testing. |
| [Guía de buenas prácticas PostgreSQL](Backend-APIs-DB/guia_buenas_practicas_postgresql.md) | SQL, diseño, constraints, índices, transacciones, vistas, mantenimiento, seguridad, backups, tuning, observabilidad y administración PostgreSQL. |
| [Instalación y configuración de PostgreSQL en Ubuntu](Backend-APIs-DB/guia_instalacion_configuracion_postgresql_ubuntu.md) | Instalación, usuarios, roles, `postgresql.conf`, `pg_hba.conf`, backups, logs, TLS, tuning, extensiones y troubleshooting. |
| [Anexo PostgreSQL para ingeniería y ciencia de datos](Backend-APIs-DB/anexo_postgresql_ingenieria_ciencia_datos.md) | ETL/ELT, data warehouses, data lakes, modelado dimensional, gobernanza, pipelines, feature engineering y búsqueda vectorial. |
| [Anexo PostgreSQL para RAG](Backend-APIs-DB/anexo_postgresql_rag.md) | RAG, embeddings, `pgvector`, JSONB, búsqueda híbrida, chunking, documentos, BETO/BERT y pipelines de ingesta. |

---

## Frontend, web design y SEO

| Guía | Tema |
|---|---|
| [Guía de buenas prácticas CSS](Frontend-WebDesign/guia_buenas_practicas_css.md) | CSS básico, intermedio y avanzado, responsividad, paletas, contratos de estilos, layouts, animaciones, drag and drop y diseño visual. |
| [Guía de buenas prácticas React](Frontend-WebDesign/guia_buenas_practicas_react.md) | React moderno, hooks, rutas, APIs, formularios, TanStack Query, Supabase, CRUD, eventos, imágenes y patrones avanzados. |
| [Guía de buenas prácticas Next.js](Frontend-WebDesign/guia_buenas_practicas_nextjs.md) | App Router, Server/Client Components, layouts, rutas, Route Handlers, Server Functions, Supabase, TanStack Query y despliegue. |
| [Guía de SEO, campañas web y adquisición digital con Next.js](Frontend-WebDesign/guia_seo_campanas_nextjs.md) | SEO técnico, keywords, metadata, sitemap, robots, Open Graph, JSON-LD, Google Ads, redes sociales, analítica y casos públicos/privados/híbridos. |

---

## Seguridad, infraestructura y operaciones

| Guía | Tema |
|---|---|
| [Guía de ciberseguridad para desarrollo web](Security-Infrastructure-Operations/guia_buenas_practicas_ciberseguridad_web.md) | Autenticación, autorización, OAuth2, cookies, backend como autoridad, SQL injection, XSS, CSRF, SSRF, auditorías y hardening. |
| [Guía de AWS para producción de sistemas web](Security-Infrastructure-Operations/guia_aws_produccion_sistemas_web.md) | VPC, subredes públicas/privadas, ALB, NAT Gateway, EC2, RDS, S3, CloudFront, Cognito, IAM, Secrets Manager, SQS, SNS, Lambda y observabilidad. |
| [Guía de agentes de IA con AWS Bedrock](Security-Infrastructure-Operations/guia_agentes_ia_aws_bedrock.md) | Agentes IA, Bedrock Agents, Knowledge Bases, Guardrails, Flows, RAG, multiagentes, LangChain, LangGraph, n8n, seguridad, evaluación y gobernanza. |
| [Guía de creación, instalación y operación de servicios en Ubuntu](Security-Infrastructure-Operations/guia_creacion_instalacion_servicios_ubuntu.md) | `systemd`, `systemctl`, `journalctl`, servicios Python, timers, recursos, hardening, monitoreo, gobernanza y troubleshooting. |
| [Guía de instalación y configuración de Apache en Ubuntu](Security-Infrastructure-Operations/guia_instalacion_configuracion_apache_ubuntu.md) | Apache, Virtual Hosts, HTTPS, Certbot, reverse proxy, PHP-FPM, logs, headers, seguridad, performance y troubleshooting. |
| [Guía de uso de conexiones SSH](Security-Infrastructure-Operations/guia_uso_conexiones_ssh.md) | OpenSSH, llaves, PEM, PuTTY, `scp`, `sftp`, `rsync`, túneles, Jupyter por SSH, ProxyJump, ControlMaster y hardening. |
| [Subir y bajar archivos con SCP](Security-Infrastructure-Operations/SubirYBajarScp.md) | Apunte breve sobre transferencia de archivos con `scp`. |
| [Jupyter en cloud](Security-Infrastructure-Operations/JupyterCloud.md) | Apunte breve sobre uso de Jupyter en servidores o entornos cloud. |

---

## Git, gestión y procesos

| Guía | Tema |
|---|---|
| [Configurar Git](Git-Management-Processes/ConfigurarGit.md) | Apunte breve de configuración inicial de Git. |
| [Guía de gestión de proyectos y productos digitales](Git-Management-Processes/guia_gestion_proyectos_productos_agil.md) | Creación de productos, contacto con clientes, presentaciones, seguimiento, derivas, marketing, Scrum, Kanban y metodologías ágiles. |

---

## Rutas de aprendizaje sugeridas

### Ruta 1: Backend Python productivo

1. [PEP 8 para Python](Python-CodeQuiality-Design/guia_pep8_python.md)
2. [QA y testing con Python](Python-CodeQuiality-Design/guia_qa_testing_python.md)
3. [FastAPI](Backend-APIs-DB/guia_buenas_practicas_fastapi_completa.md)
4. [PostgreSQL](Backend-APIs-DB/guia_buenas_practicas_postgresql.md)
5. [Instalación de PostgreSQL en Ubuntu](Backend-APIs-DB/guia_instalacion_configuracion_postgresql_ubuntu.md)
6. [Servicios en Ubuntu](Security-Infrastructure-Operations/guia_creacion_instalacion_servicios_ubuntu.md)
7. [Apache en Ubuntu](Security-Infrastructure-Operations/guia_instalacion_configuracion_apache_ubuntu.md)
8. [SSH](Security-Infrastructure-Operations/guia_uso_conexiones_ssh.md)

### Ruta 2: Frontend moderno y adquisición digital

1. [CSS](Frontend-WebDesign/guia_buenas_practicas_css.md)
2. [React](Frontend-WebDesign/guia_buenas_practicas_react.md)
3. [Next.js](Frontend-WebDesign/guia_buenas_practicas_nextjs.md)
4. [SEO y campañas con Next.js](Frontend-WebDesign/guia_seo_campanas_nextjs.md)
5. [Ciberseguridad web](Security-Infrastructure-Operations/guia_buenas_practicas_ciberseguridad_web.md)
6. [QA y testing](Python-CodeQuiality-Design/guia_qa_testing_python.md)

### Ruta 3: Datos, PostgreSQL y RAG

1. [PostgreSQL base](Backend-APIs-DB/guia_buenas_practicas_postgresql.md)
2. [PostgreSQL para ingeniería y ciencia de datos](Backend-APIs-DB/anexo_postgresql_ingenieria_ciencia_datos.md)
3. [PostgreSQL para RAG](Backend-APIs-DB/anexo_postgresql_rag.md)
4. [Instalación de PostgreSQL en Ubuntu](Backend-APIs-DB/guia_instalacion_configuracion_postgresql_ubuntu.md)
5. [Agentes de IA con AWS Bedrock](Security-Infrastructure-Operations/guia_agentes_ia_aws_bedrock.md)
6. [AWS para producción web](Security-Infrastructure-Operations/guia_aws_produccion_sistemas_web.md)

### Ruta 4: Producción, servidores y operación

1. [SSH](Security-Infrastructure-Operations/guia_uso_conexiones_ssh.md)
2. [Servicios en Ubuntu](Security-Infrastructure-Operations/guia_creacion_instalacion_servicios_ubuntu.md)
3. [Apache en Ubuntu](Security-Infrastructure-Operations/guia_instalacion_configuracion_apache_ubuntu.md)
4. [PostgreSQL en Ubuntu](Backend-APIs-DB/guia_instalacion_configuracion_postgresql_ubuntu.md)
5. [AWS para producción web](Security-Infrastructure-Operations/guia_aws_produccion_sistemas_web.md)
6. [Ciberseguridad web](Security-Infrastructure-Operations/guia_buenas_practicas_ciberseguridad_web.md)
7. [QA y testing](Python-CodeQuiality-Design/guia_qa_testing_python.md)

### Ruta 5: IA aplicada, agentes y automatización

1. [PostgreSQL para RAG](Backend-APIs-DB/anexo_postgresql_rag.md)
2. [Agentes de IA con AWS Bedrock](Security-Infrastructure-Operations/guia_agentes_ia_aws_bedrock.md)
3. [AWS para producción web](Security-Infrastructure-Operations/guia_aws_produccion_sistemas_web.md)
4. [Ciberseguridad web](Security-Infrastructure-Operations/guia_buenas_practicas_ciberseguridad_web.md)
5. [QA y testing](Python-CodeQuiality-Design/guia_qa_testing_python.md)
6. [Servicios en Ubuntu](Security-Infrastructure-Operations/guia_creacion_instalacion_servicios_ubuntu.md)

### Ruta 6: Gestión, producto y entrega

1. [Gestión de proyectos y productos digitales](Git-Management-Processes/guia_gestion_proyectos_productos_agil.md)
2. [SEO y campañas con Next.js](Frontend-WebDesign/guia_seo_campanas_nextjs.md)
3. [QA y testing](Python-CodeQuiality-Design/guia_qa_testing_python.md)
4. [Ciberseguridad web](Security-Infrastructure-Operations/guia_buenas_practicas_ciberseguridad_web.md)
5. [AWS para producción web](Security-Infrastructure-Operations/guia_aws_produccion_sistemas_web.md)

---

## Convenciones del repositorio

Los archivos están en formato Markdown (`.md`) para facilitar lectura directa desde GitHub, GitLab, VS Code o cualquier visor Markdown.

Convenciones recomendadas:

```text
- Mantener una guía por tema principal.
- Usar nombres descriptivos.
- Agregar nuevas guías dentro de la carpeta temática correspondiente.
- Actualizar este README al agregar, mover o renombrar archivos.
- Usar anexos cuando una guía extiende una guía base.
- Evitar duplicar contenido entre guías.
- Probar comandos antes de ejecutarlos en producción.
```

---

## Estado de las guías

| Tipo | Estado |
|---|---|
| Guías largas | Material de referencia y estudio. |
| Apuntes breves | Notas rápidas reutilizables. |
| Anexos | Extensiones especializadas de una guía base. |

---

## Uso recomendado

Para estudiar:

```text
1. Elegir una ruta de aprendizaje.
2. Leer la guía principal.
3. Ejecutar ejemplos en un entorno local.
4. Adaptar checklists a proyectos reales.
5. Registrar mejoras o temas pendientes como issues.
```

Para consultar rápidamente:

```text
- Usar la búsqueda del editor.
- Buscar por comandos.
- Ir directo a checklists.
- Copiar plantillas y ajustarlas al caso real.
```

Para producción:

```text
- Validar comandos en staging antes de producción.
- Revisar seguridad, permisos y secretos.
- Adaptar configuraciones a la infraestructura real.
- Documentar cambios.
```

---

## Sugerencias de mejora futura

```text
[ ] Corregir el nombre de la carpeta Python-CodeQuiality-Design si se decide estandarizarlo.
[ ] Agregar índice automático por headings.
[ ] Agregar ejemplos ejecutables por guía.
[ ] Agregar diagramas de arquitectura.
[ ] Agregar scripts complementarios.
[ ] Agregar una licencia.
[ ] Agregar CONTRIBUTING.md si más personas colaboran.
[ ] Agregar CHANGELOG.md.
[ ] Publicar como sitio estático con MkDocs, Docusaurus, Nextra o GitHub Pages.
```

---

## Licencia

Pendiente de definir.

Si este repositorio será público, conviene agregar una licencia explícita, por ejemplo MIT, Apache 2.0, Creative Commons o una licencia privada según el uso esperado.

---

## Autoría

Repositorio de tutoriales técnicos preparado como material de estudio, consulta y apoyo para proyectos de software, datos, infraestructura, seguridad, IA y gestión de productos.
