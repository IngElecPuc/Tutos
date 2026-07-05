# Tutos

Repositorio personal de tutoriales técnicos, guías prácticas y apuntes de referencia para desarrollo de software, datos, infraestructura, QA, seguridad, frontend, backend, administración de servidores y gestión de proyectos.

El objetivo de este repositorio es centralizar material de estudio y consulta rápida, con guías escritas en español y orientadas a aplicación práctica. La mayoría de los documentos combinan fundamentos, buenas prácticas, ejemplos de código, comandos útiles, checklists y recomendaciones de operación.

---

## Contenido

### Python, calidad de código y diseño

| Guía | Tema |
|---|---|
| [Guía práctica de PEP 8 para Python](guia_pep8_python.md) | Estilo Python, naming, imports, docstrings, type hints, linters, formatters y configuración con Ruff/Black/isort. |
| [Guía de QA y testing de sistemas con Python](guia_qa_testing_python.md) | Unit tests, integration tests, smoke tests, debugging, logging, coverage, CI/CD, monitoreo y testing avanzado. |
| [Guía de los 23 patrones de diseño](guia_23_patrones_diseno_python_c_cpp_csharp.md) | Patrones GoF creacionales, estructurales y de comportamiento, con ejemplos en Python, C, C++ y C#. |
| [Comandos básicos de `breakpoint()` en Python](breakpointPython.md) | Uso práctico de `breakpoint()` y depuración interactiva en Python. |

---

### Backend, APIs y bases de datos

| Guía | Tema |
|---|---|
| [Guía de buenas prácticas para APIs con FastAPI](guia_buenas_practicas_fastapi_completa.md) | FastAPI, validaciones, modelos, errores, dependencias, OAuth2, SQLModel, middlewares, templates y testing. |
| [Guía de buenas prácticas PostgreSQL](guia_buenas_practicas_postgresql.md) | Diseño, SQL, índices, mantenimiento, seguridad, backups, transacciones, tuning y administración PostgreSQL. |
| [Instalación y configuración de PostgreSQL en Ubuntu](guia_instalacion_configuracion_postgresql_ubuntu.md) | Instalación, usuarios, roles, `pg_hba.conf`, backups, logs, TLS, tuning y troubleshooting. |
| [Anexo PostgreSQL para ingeniería y ciencia de datos](anexo_postgresql_ingenieria_ciencia_datos.md) | ETL/ELT, data warehouses, data lakes, modelado dimensional, gobernanza, feature engineering y vector search. |
| [Anexo PostgreSQL para RAG](anexo_postgresql_rag.md) | RAG, embeddings, `pgvector`, JSONB, búsqueda híbrida, chunking, documentos, BETO/BERT y pipelines de ingesta. |

---

### Frontend y desarrollo web

| Guía | Tema |
|---|---|
| [Guía de buenas prácticas CSS](guia_buenas_practicas_css.md) | CSS básico, intermedio y avanzado, responsividad, paletas, contratos de estilos, layouts, animaciones y drag and drop. |
| [Guía de buenas prácticas React](guia_buenas_practicas_react.md) | React moderno, hooks, rutas, APIs, formularios, TanStack Query, Supabase, CRUD, eventos y patrones avanzados. |
| [Guía de buenas prácticas Next.js](guia_buenas_practicas_nextjs.md) | App Router, Server/Client Components, rutas, layouts, Route Handlers, Server Functions, Supabase y producción. |

---

### Seguridad, infraestructura y operación

| Guía | Tema |
|---|---|
| [Guía de ciberseguridad para desarrollo web](guia_buenas_practicas_ciberseguridad_web.md) | Autenticación, autorización, OAuth2, cookies, backend como autoridad, SQL injection, XSS, CSRF, auditorías y hardening. |
| [Guía de AWS para producción de sistemas web](guia_aws_produccion_sistemas_web.md) | VPC, subredes, ALB, NAT Gateway, EC2, RDS, S3, CloudFront, Cognito, IAM, Secrets Manager, SQS, SNS, Lambda y observabilidad. |
| [Guía de creación, instalación y operación de servicios en Ubuntu](guia_creacion_instalacion_servicios_ubuntu.md) | `systemd`, `systemctl`, `journalctl`, servicios Python, recursos, hardening, timers, monitoreo y gobernanza. |
| [Guía de instalación y configuración de Apache en Ubuntu](guia_instalacion_configuracion_apache_ubuntu.md) | Apache, Virtual Hosts, HTTPS, Certbot, reverse proxy, PHP-FPM, logs, headers, performance y troubleshooting. |
| [Guía de uso de conexiones SSH](guia_uso_conexiones_ssh.md) | OpenSSH, llaves, PEM, PuTTY, `scp`, `sftp`, túneles, Jupyter por SSH, ProxyJump, ControlMaster y hardening. |
| [Subir y bajar archivos con SCP](SubirYBajarScp.md) | Apunte breve sobre transferencia de archivos con `scp`. |
| [Jupyter en cloud](JupyterCloud.md) | Apunte breve sobre uso de Jupyter en servidores o entornos cloud. |

---

### Git, gestión y procesos

| Guía | Tema |
|---|---|
| [Configurar Git](ConfigurarGit.md) | Apunte breve de configuración inicial de Git. |
| [Guía de gestión de proyectos y productos digitales](guia_gestion_proyectos_productos_agil.md) | Creación de productos, contacto con clientes, presentaciones, seguimiento, derivas, marketing, Scrum y metodologías ágiles. |

---

## Rutas de aprendizaje sugeridas

### Ruta 1: Backend Python productivo

1. [PEP 8 para Python](guia_pep8_python.md)
2. [QA y testing con Python](guia_qa_testing_python.md)
3. [FastAPI](guia_buenas_practicas_fastapi_completa.md)
4. [PostgreSQL](guia_buenas_practicas_postgresql.md)
5. [Instalación de PostgreSQL en Ubuntu](guia_instalacion_configuracion_postgresql_ubuntu.md)
6. [Servicios en Ubuntu](guia_creacion_instalacion_servicios_ubuntu.md)
7. [Apache en Ubuntu](guia_instalacion_configuracion_apache_ubuntu.md)
8. [SSH](guia_uso_conexiones_ssh.md)

### Ruta 2: Frontend moderno

1. [CSS](guia_buenas_practicas_css.md)
2. [React](guia_buenas_practicas_react.md)
3. [Next.js](guia_buenas_practicas_nextjs.md)
4. [Ciberseguridad web](guia_buenas_practicas_ciberseguridad_web.md)
5. [QA y testing](guia_qa_testing_python.md)

### Ruta 3: Datos, PostgreSQL y RAG

1. [PostgreSQL base](guia_buenas_practicas_postgresql.md)
2. [PostgreSQL para ingeniería y ciencia de datos](anexo_postgresql_ingenieria_ciencia_datos.md)
3. [PostgreSQL para RAG](anexo_postgresql_rag.md)
4. [Instalación de PostgreSQL en Ubuntu](guia_instalacion_configuracion_postgresql_ubuntu.md)
5. [AWS para sistemas web](guia_aws_produccion_sistemas_web.md)

### Ruta 4: Producción, servidores y operación

1. [SSH](guia_uso_conexiones_ssh.md)
2. [Servicios en Ubuntu](guia_creacion_instalacion_servicios_ubuntu.md)
3. [Apache en Ubuntu](guia_instalacion_configuracion_apache_ubuntu.md)
4. [PostgreSQL en Ubuntu](guia_instalacion_configuracion_postgresql_ubuntu.md)
5. [AWS para producción web](guia_aws_produccion_sistemas_web.md)
6. [Ciberseguridad web](guia_buenas_practicas_ciberseguridad_web.md)
7. [QA y testing](guia_qa_testing_python.md)

### Ruta 5: Gestión, producto y entrega

1. [Gestión de proyectos y productos digitales](guia_gestion_proyectos_productos_agil.md)
2. [QA y testing](guia_qa_testing_python.md)
3. [Ciberseguridad web](guia_buenas_practicas_ciberseguridad_web.md)
4. [AWS para producción web](guia_aws_produccion_sistemas_web.md)

---

## Convenciones del repositorio

Los archivos están en formato Markdown (`.md`) para que puedan leerse directamente desde GitHub, GitLab, VS Code o cualquier visor Markdown.

Convenciones recomendadas:

```text
- Mantener nombres descriptivos.
- Usar minúsculas y guiones bajos para nuevas guías.
- Mantener una guía por tema principal.
- Agregar enlaces nuevos a este README.
- Evitar duplicar contenido; preferir anexos cuando el tema extiende una guía existente.
- Revisar comandos antes de ejecutarlos en producción.
```

---

## Estado de las guías

| Tipo | Estado |
|---|---|
| Guías largas | Material de referencia y estudio. |
| Apuntes breves | Notas rápidas reutilizables. |
| Anexos | Extensiones especializadas de una guía base. |

Este repositorio puede crecer como biblioteca personal. Una mejora futura recomendable es reorganizar las guías en carpetas temáticas, por ejemplo:

```text
python/
backend/
frontend/
database/
infraestructura/
seguridad/
gestion/
apuntes/
```

Por ahora, los enlaces están preparados para funcionar con la estructura actual en la raíz del repositorio.

---

## Uso recomendado

Para estudiar:

```text
1. Elegir una ruta de aprendizaje.
2. Leer la guía principal.
3. Ejecutar ejemplos en un entorno local.
4. Adaptar checklists a proyectos reales.
5. Registrar dudas o mejoras como issues.
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
[ ] Crear carpetas temáticas.
[ ] Agregar índice automático por headings.
[ ] Agregar ejemplos ejecutables por guía.
[ ] Agregar diagramas de arquitectura.
[ ] Agregar scripts complementarios.
[ ] Agregar una licencia.
[ ] Agregar CONTRIBUTING.md si más personas colaboran.
[ ] Agregar CHANGELOG.md.
[ ] Publicar como sitio estático con MkDocs, Docusaurus o GitHub Pages.
```

---

## Licencia

Pendiente de definir.

Si este repositorio será público, conviene agregar una licencia explícita, por ejemplo MIT, Apache 2.0, Creative Commons o una licencia privada según el uso esperado.

---

## Autoría

Repositorio de tutoriales técnicos preparado como material de estudio, consulta y apoyo para proyectos de software, datos e infraestructura.
