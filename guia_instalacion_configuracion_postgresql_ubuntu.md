# Guía de instalación y configuración de PostgreSQL en Ubuntu

## Objetivo

Esta guía explica cómo instalar, configurar, asegurar y operar PostgreSQL en Ubuntu. Está pensada para desarrollo local, servidores de testing/staging y bases productivas pequeñas o medianas administradas directamente en una máquina Ubuntu.

Está organizada en tres niveles:

```text
1. Básico: instalación, servicio, psql, usuarios, bases de datos y comandos esenciales.
2. Intermedio: configuración, acceso remoto controlado, autenticación, firewall, backups, logs y mantenimiento.
3. Avanzado: hardening, tuning inicial, monitoreo, extensiones, múltiples versiones, upgrades y troubleshooting.
```

La guía asume Ubuntu Server o Ubuntu Desktop moderno. Los comandos usan `sudo`.

---

# Parte I: conceptos básicos

## 1. PostgreSQL en Ubuntu

PostgreSQL es un sistema de base de datos relacional de código abierto. En Ubuntu se puede instalar de dos formas principales:

```text
1. Repositorios oficiales de Ubuntu.
2. Repositorio APT oficial del PostgreSQL Global Development Group, PGDG.
```

### Opción A: repositorios de Ubuntu

Ventajas:

```text
Más simple.
Integrado con el ciclo de vida de Ubuntu.
Suficiente para desarrollo local y muchos servidores internos.
```

Desventajas:

```text
La versión disponible depende de la versión de Ubuntu.
No siempre tendrás la última versión mayor de PostgreSQL.
```

Instalación típica:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

### Opción B: repositorio oficial PGDG

Ventajas:

```text
Permite instalar versiones específicas.
Permite acceder a versiones actuales de PostgreSQL.
Incluye paquetes de extensiones y herramientas adicionales.
```

Desventajas:

```text
Agrega un repositorio externo al de Ubuntu.
Debes administrar con más claridad qué versión instalas.
```

Instalación típica:

```bash
sudo apt install -y postgresql-common ca-certificates
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
sudo apt update
sudo apt install postgresql-18 postgresql-contrib-18
```

Regla práctica:

```text
Para aprender o desarrollo local: repositorio Ubuntu está bien.
Para producción o versión específica: PGDG suele ser mejor.
```

---

## 2. Verificar versión de Ubuntu

Antes de instalar, revisa tu versión:

```bash
lsb_release -a
```

O:

```bash
cat /etc/os-release
```

Salida esperada, ejemplo:

```text
NAME="Ubuntu"
VERSION="24.04 LTS (Noble Numbat)"
VERSION_CODENAME=noble
```

El `VERSION_CODENAME` importa si configuras manualmente el repositorio PGDG.

---

## 3. Paquetes importantes

Paquetes frecuentes:

```text
postgresql
    metapaquete del servidor PostgreSQL disponible en el repositorio configurado.

postgresql-client
    herramientas cliente como psql.

postgresql-contrib
    extensiones adicionales comunes.

postgresql-common
    herramientas Debian/Ubuntu para administrar clusters PostgreSQL.

libpq-dev
    headers y librerías para compilar clientes Python, C, etc.

postgresql-18
    versión específica de PostgreSQL desde PGDG, si está disponible.

postgresql-client-18
    cliente específico de versión.

postgresql-contrib-18
    extensiones contrib para esa versión.
```

Para aplicaciones Python que usan `psycopg`, a veces necesitas:

```bash
sudo apt install libpq-dev python3-dev build-essential
```

---

## 4. Qué instala Ubuntu

En Ubuntu/Debian, PostgreSQL se administra por clusters.

Un cluster PostgreSQL es una instancia concreta de PostgreSQL con:

```text
Versión.
Nombre.
Puerto.
Directorio de datos.
Archivos de configuración.
Estado del servicio.
```

El cluster por defecto suele llamarse:

```text
main
```

Ejemplo:

```text
Versión: 18
Cluster: main
Puerto: 5432
```

Comando útil:

```bash
pg_lsclusters
```

Salida típica:

```text
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5432 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
```

---

# Parte II: instalación básica

## 5. Instalación desde repositorios de Ubuntu

Esta es la forma más simple.

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

Verifica instalación:

```bash
psql --version
```

Verifica servicio:

```bash
sudo systemctl status postgresql
```

Verifica clusters:

```bash
pg_lsclusters
```

Entrar a PostgreSQL como usuario administrativo local:

```bash
sudo -u postgres psql
```

Salir de `psql`:

```sql
\q
```

---

## 6. Instalación desde PGDG

Usa esta opción si necesitas una versión específica.

### 6.1 Configuración automática recomendada

```bash
sudo apt update
sudo apt install -y postgresql-common ca-certificates
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
sudo apt update
```

Instalar versión específica:

```bash
sudo apt install postgresql-18 postgresql-client-18 postgresql-contrib-18
```

Verificar:

```bash
psql --version
pg_lsclusters
```

---

### 6.2 Configuración manual del repositorio PGDG

Usa esto si prefieres controlar el archivo de repositorio manualmente.

Instala dependencias:

```bash
sudo apt update
sudo apt install -y curl ca-certificates
```

Crea directorio para la llave:

```bash
sudo install -d /usr/share/postgresql-common/pgdg
```

Descarga la llave del repositorio:

```bash
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc \
    --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
```

Crea archivo de repositorio usando el codename de Ubuntu:

```bash
. /etc/os-release

sudo tee /etc/apt/sources.list.d/pgdg.sources > /dev/null <<EOF
Types: deb
URIs: https://apt.postgresql.org/pub/repos/apt
Suites: ${VERSION_CODENAME}-pgdg
Architectures: $(dpkg --print-architecture)
Components: main
Signed-By: /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc
EOF
```

Actualiza:

```bash
sudo apt update
```

Instala:

```bash
sudo apt install postgresql-18 postgresql-client-18 postgresql-contrib-18
```

---

## 7. Verificar que PostgreSQL está activo

Servicio general:

```bash
sudo systemctl status postgresql
```

Ver clusters:

```bash
pg_lsclusters
```

Ver si PostgreSQL acepta conexiones:

```bash
pg_isready
```

Especificando host y puerto:

```bash
pg_isready -h localhost -p 5432
```

Salida posible:

```text
localhost:5432 - accepting connections
```

---

## 8. Comandos de servicio

Con `systemctl`:

```bash
sudo systemctl start postgresql
sudo systemctl stop postgresql
sudo systemctl restart postgresql
sudo systemctl reload postgresql
sudo systemctl status postgresql
```

Con `pg_ctlcluster`, útil en Ubuntu/Debian cuando hay versiones o clusters específicos:

```bash
sudo pg_ctlcluster 18 main start
sudo pg_ctlcluster 18 main stop
sudo pg_ctlcluster 18 main restart
sudo pg_ctlcluster 18 main reload
sudo pg_ctlcluster 18 main status
```

Diferencia:

```text
systemctl postgresql:
Controla el servicio general.

pg_ctlcluster:
Controla un cluster específico por versión y nombre.
```

---

## 9. Archivos y directorios importantes

En Ubuntu, los archivos suelen estar aquí:

```text
Configuración:
    /etc/postgresql/<version>/main/

Archivo principal:
    /etc/postgresql/<version>/main/postgresql.conf

Autenticación:
    /etc/postgresql/<version>/main/pg_hba.conf

Ident mapping:
    /etc/postgresql/<version>/main/pg_ident.conf

Datos:
    /var/lib/postgresql/<version>/main/

Logs:
    /var/log/postgresql/postgresql-<version>-main.log
```

Ejemplo para PostgreSQL 18:

```text
/etc/postgresql/18/main/postgresql.conf
/etc/postgresql/18/main/pg_hba.conf
/var/lib/postgresql/18/main/
/var/log/postgresql/postgresql-18-main.log
```

Para conocer ubicación desde SQL:

```sql
SHOW config_file;
SHOW hba_file;
SHOW data_directory;
SHOW log_directory;
```

---

# Parte III: uso inicial con psql

## 10. Entrar a PostgreSQL

El instalador crea un usuario del sistema llamado `postgres` y un rol de base de datos llamado `postgres`.

Entrar como administrador local:

```bash
sudo -u postgres psql
```

Ver versión:

```sql
SELECT version();
```

Ver conexión actual:

```sql
\conninfo
```

Salir:

```sql
\q
```

---

## 11. Comandos útiles de psql

```sql
\l              -- listar bases de datos
\c dbname       -- conectarse a una base de datos
\dt             -- listar tablas
\d table_name   -- describir tabla
\du             -- listar roles
\dn             -- listar schemas
\dx             -- listar extensiones
\timing         -- activar medición de tiempo
\x              -- salida expandida
\q              -- salir
```

Ejemplo:

```sql
\l
\du
\c postgres
SELECT current_database(), current_user;
```

---

## 12. Crear un usuario de aplicación

No uses el superusuario `postgres` desde tu aplicación.

Crear rol de aplicación:

```sql
CREATE ROLE app_user
WITH
    LOGIN
    PASSWORD 'change_this_password';
```

Crear base de datos con dueño:

```sql
CREATE DATABASE app_db
OWNER app_user;
```

Conectarse:

```bash
psql -U app_user -d app_db -h localhost
```

Si la autenticación local está configurada con `peer`, puede fallar al usar `-U app_user` desde un usuario Linux distinto. Más adelante se explica `pg_hba.conf`.

---

## 13. Crear schema y permisos básicos

Entrar como `postgres`:

```bash
sudo -u postgres psql
```

Crear base y usuario:

```sql
CREATE ROLE app_user
WITH
    LOGIN
    PASSWORD 'change_this_password';

CREATE DATABASE app_db
OWNER app_user;
```

Conectarse a la base:

```sql
\c app_db
```

Crear schema:

```sql
CREATE SCHEMA app AUTHORIZATION app_user;
```

Dar permisos explícitos si quieres usar `public`:

```sql
GRANT USAGE ON SCHEMA public TO app_user;
GRANT CREATE ON SCHEMA public TO app_user;
```

Regla práctica:

```text
Para aplicaciones nuevas, considera usar un schema propio como app.
Evita dar permisos amplios sin necesidad.
```

---

## 14. Crear tabla de prueba

Conéctate como `app_user`:

```bash
psql -U app_user -d app_db -h localhost
```

Crear tabla:

```sql
CREATE TABLE tasks (
    id BIGSERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    completed BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Insertar:

```sql
INSERT INTO tasks (title)
VALUES ('Instalar PostgreSQL en Ubuntu');
```

Consultar:

```sql
SELECT *
FROM tasks;
```

---

# Parte IV: autenticación y acceso local

## 15. Autenticación peer

En Ubuntu, las conexiones locales por socket suelen usar `peer`.

Eso significa:

```text
El usuario Linux debe coincidir con el rol PostgreSQL.
```

Ejemplo:

```bash
sudo -u postgres psql
```

Funciona porque:

```text
Usuario Linux: postgres
Rol PostgreSQL: postgres
```

Si intentas:

```bash
psql -U app_user -d app_db
```

puede fallar si el usuario Linux actual no se llama `app_user`.

---

## 16. Autenticación por contraseña local

Para conectarte con contraseña desde localhost:

```bash
psql -U app_user -d app_db -h localhost
```

Al usar `-h localhost`, normalmente fuerzas conexión TCP en vez de socket Unix.

Para que funcione, debe existir una regla compatible en `pg_hba.conf`, por ejemplo:

```text
host    all    all    127.0.0.1/32    scram-sha-256
host    all    all    ::1/128         scram-sha-256
```

Después de editar `pg_hba.conf`:

```bash
sudo pg_ctlcluster 18 main reload
```

O:

```bash
sudo systemctl reload postgresql
```

---

## 17. Cambiar contraseña de un rol

Desde `psql`:

```sql
ALTER ROLE app_user WITH PASSWORD 'new_secure_password';
```

Cambiar contraseña de `postgres`:

```sql
ALTER ROLE postgres WITH PASSWORD 'new_secure_password';
```

Advertencia:

```text
No uses contraseñas débiles.
No guardes contraseñas en scripts versionados.
Para producción usa un gestor de secretos.
```

---

## 18. Configurar SCRAM-SHA-256

SCRAM-SHA-256 es el método recomendado para autenticación con contraseña.

En `postgresql.conf`:

```conf
password_encryption = scram-sha-256
```

Después, cuando cambies o crees contraseñas, se almacenarán usando SCRAM.

```sql
ALTER ROLE app_user WITH PASSWORD 'new_secure_password';
```

En `pg_hba.conf`:

```text
host    app_db    app_user    127.0.0.1/32    scram-sha-256
```

Recargar:

```bash
sudo pg_ctlcluster 18 main reload
```

---

# Parte V: configuración principal

## 19. postgresql.conf

Archivo:

```bash
sudo nano /etc/postgresql/18/main/postgresql.conf
```

Parámetros frecuentes:

```conf
listen_addresses = 'localhost'
port = 5432
max_connections = 100
shared_buffers = 1GB
effective_cache_size = 3GB
work_mem = 16MB
maintenance_work_mem = 256MB
log_min_duration_statement = 500
```

No copies tuning de internet sin medir. Estos valores dependen de:

```text
RAM.
CPU.
Disco.
Cantidad de conexiones.
Patrón de consultas.
Tamaño de datos.
Carga de escritura.
Carga de lectura.
```

---

## 20. Recargar vs reiniciar

Algunos parámetros se aplican con reload. Otros requieren restart.

Recargar:

```bash
sudo pg_ctlcluster 18 main reload
```

Reiniciar:

```bash
sudo pg_ctlcluster 18 main restart
```

Desde SQL:

```sql
SELECT pg_reload_conf();
```

Ver si hay configuraciones pendientes de reinicio:

```sql
SELECT name, setting, pending_restart
FROM pg_settings
WHERE pending_restart = true;
```

Regla:

```text
Usa reload cuando baste.
Usa restart solo cuando sea necesario.
Planifica restart en producción.
```

---

## 21. Ver configuración activa

Desde `psql`:

```sql
SHOW listen_addresses;
SHOW port;
SHOW max_connections;
SHOW shared_buffers;
SHOW data_directory;
SHOW config_file;
SHOW hba_file;
```

Ver parámetros relevantes:

```sql
SELECT name, setting, unit, context
FROM pg_settings
WHERE name IN (
    'listen_addresses',
    'port',
    'max_connections',
    'shared_buffers',
    'work_mem',
    'maintenance_work_mem',
    'effective_cache_size'
);
```

---

# Parte VI: acceso remoto controlado

## 22. Cuándo permitir acceso remoto

Permite acceso remoto solo si tienes una razón clara:

```text
Aplicación en otro servidor.
Herramienta BI interna.
Administración desde red privada.
Servidor de staging.
```

Evita:

```text
Exponer PostgreSQL directamente a internet.
Abrir puerto 5432 a 0.0.0.0/0.
Usar usuario postgres remotamente.
```

Mejor:

```text
Acceso por red privada.
VPN.
Bastion.
SSH tunnel.
Security groups en cloud.
Firewall restrictivo.
TLS.
Usuarios con permisos mínimos.
```

---

## 23. Configurar listen_addresses

Archivo:

```bash
sudo nano /etc/postgresql/18/main/postgresql.conf
```

Local solamente:

```conf
listen_addresses = 'localhost'
```

Escuchar en una IP privada concreta:

```conf
listen_addresses = '10.0.1.10'
```

Escuchar en todas las interfaces:

```conf
listen_addresses = '*'
```

Recomendación:

```text
Prefiere IP privada concreta.
Usa '*' solo si controlas bien firewall y pg_hba.conf.
```

`listen_addresses` requiere restart:

```bash
sudo pg_ctlcluster 18 main restart
```

---

## 24. Configurar pg_hba.conf

Archivo:

```bash
sudo nano /etc/postgresql/18/main/pg_hba.conf
```

Formato general:

```text
TYPE    DATABASE    USER    ADDRESS         METHOD
```

Ejemplo local:

```text
local   all         postgres                peer
local   all         all                     peer
host    all         all     127.0.0.1/32    scram-sha-256
host    all         all     ::1/128         scram-sha-256
```

Permitir app desde una red privada:

```text
hostssl app_db      app_user    10.0.10.0/24    scram-sha-256
```

Permitir solo una IP:

```text
hostssl app_db      app_user    10.0.10.25/32   scram-sha-256
```

Evita:

```text
host    all         all         0.0.0.0/0        trust
host    all         all         0.0.0.0/0        md5
```

Después de editar:

```bash
sudo pg_ctlcluster 18 main reload
```

Ver errores de reglas:

```sql
SELECT *
FROM pg_hba_file_rules
WHERE error IS NOT NULL;
```

---

## 25. Firewall con UFW

Activar UFW:

```bash
sudo ufw enable
```

Permitir SSH antes de activar si estás conectado por SSH:

```bash
sudo ufw allow OpenSSH
```

Permitir PostgreSQL solo desde una IP o red privada:

```bash
sudo ufw allow from 10.0.10.25 to any port 5432 proto tcp
```

O desde una red:

```bash
sudo ufw allow from 10.0.10.0/24 to any port 5432 proto tcp
```

Ver reglas:

```bash
sudo ufw status verbose
```

No recomendado:

```bash
sudo ufw allow 5432/tcp
```

Eso puede abrir PostgreSQL más de lo necesario.

---

## 26. Probar conexión remota

Desde otro host:

```bash
psql -h 10.0.1.10 -p 5432 -U app_user -d app_db
```

Con string de conexión:

```bash
psql "postgresql://app_user@10.0.1.10:5432/app_db"
```

Probar disponibilidad:

```bash
pg_isready -h 10.0.1.10 -p 5432
```

Si falla, revisar:

```text
1. PostgreSQL escucha en la IP correcta.
2. pg_hba.conf permite la conexión.
3. Firewall permite puerto 5432.
4. Red enruta correctamente.
5. Usuario y contraseña son correctos.
6. El servicio está online.
```

---

# Parte VII: TLS/SSL

## 27. Por qué usar TLS

TLS cifra la conexión cliente-servidor.

Úsalo si:

```text
La conexión cruza redes no confiables.
Hay datos sensibles.
Hay acceso remoto.
Hay requisitos de compliance.
```

Para conexiones locales dentro de la misma máquina puede ser menos crítico, pero en producción remota suele ser recomendable.

---

## 28. Ver si SSL está activo

Desde `psql`:

```sql
SHOW ssl;
```

Ver conexión actual:

```sql
SELECT ssl
FROM pg_stat_ssl
WHERE pid = pg_backend_pid();
```

---

## 29. Configuración básica de SSL

En `postgresql.conf`:

```conf
ssl = on
ssl_cert_file = '/etc/ssl/certs/postgresql-server.crt'
ssl_key_file = '/etc/ssl/private/postgresql-server.key'
```

La clave privada debe ser protegida:

```bash
sudo chown postgres:postgres /etc/ssl/private/postgresql-server.key
sudo chmod 600 /etc/ssl/private/postgresql-server.key
```

En `pg_hba.conf`, exige SSL:

```text
hostssl app_db app_user 10.0.10.0/24 scram-sha-256
```

Reinicia:

```bash
sudo pg_ctlcluster 18 main restart
```

---

# Parte VIII: backups y restauración

## 30. Backup lógico con pg_dump

Backup en formato custom:

```bash
pg_dump -U app_user -h localhost -d app_db -Fc -f app_db.dump
```

Ventajas del formato custom:

```text
Flexible.
Compatible con pg_restore.
Permite seleccionar objetos.
Permite restauración paralela en ciertos casos.
```

Backup con timestamp:

```bash
mkdir -p ~/backups

pg_dump -U app_user -h localhost -d app_db -Fc \
    -f ~/backups/app_db_$(date +%Y%m%d_%H%M%S).dump
```

---

## 31. Restaurar con pg_restore

Crear base destino:

```bash
createdb -U postgres -h localhost app_db_restore
```

Restaurar:

```bash
pg_restore -U postgres -h localhost -d app_db_restore app_db.dump
```

Restaurar limpiando objetos existentes:

```bash
pg_restore -U postgres -h localhost \
    --clean \
    --if-exists \
    -d app_db_restore \
    app_db.dump
```

---

## 32. Backup SQL plano

Crear:

```bash
pg_dump -U app_user -h localhost -d app_db > app_db.sql
```

Restaurar:

```bash
psql -U postgres -h localhost -d app_db_restore < app_db.sql
```

Uso recomendado:

```text
Migraciones simples.
Inspección manual.
Bases pequeñas.
```

Para producción, suele ser preferible `-Fc`.

---

## 33. Backup de todos los roles y globals

`pg_dump` no respalda roles globales por sí solo.

Respaldar globals:

```bash
pg_dumpall -U postgres -h localhost --globals-only > globals.sql
```

Restaurar:

```bash
psql -U postgres -h localhost -f globals.sql
```

Respaldar todo el cluster:

```bash
pg_dumpall -U postgres -h localhost > full_cluster.sql
```

---

## 34. Backups físicos y PITR

Para producción seria, `pg_dump` puede no ser suficiente.

Estrategias:

```text
pg_dump:
Backup lógico.

pg_basebackup:
Backup físico base.

WAL archiving:
Permite Point-in-Time Recovery.

Herramientas:
pgBackRest, Barman, WAL-G.
```

Regla:

```text
Si necesitas recuperar a un punto específico en el tiempo,
diseña PITR con backups físicos y WAL archiving.
```

---

## 35. Automatizar backup simple con cron

Script:

```bash
sudo nano /usr/local/bin/backup_app_db.sh
```

Contenido:

```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/var/backups/postgresql"
DB_NAME="app_db"
DB_USER="postgres"
TIMESTAMP="$(date +%Y%m%d_%H%M%S)"

mkdir -p "$BACKUP_DIR"

pg_dump -U "$DB_USER" -d "$DB_NAME" -Fc \
    -f "$BACKUP_DIR/${DB_NAME}_${TIMESTAMP}.dump"

find "$BACKUP_DIR" -name "${DB_NAME}_*.dump" -mtime +14 -delete
```

Permisos:

```bash
sudo chmod +x /usr/local/bin/backup_app_db.sh
```

Cron:

```bash
sudo crontab -e
```

Ejemplo diario a las 02:30:

```cron
30 2 * * * /usr/local/bin/backup_app_db.sh >> /var/log/postgresql/backup_app_db.log 2>&1
```

Advertencias:

```text
Prueba restauraciones.
No guardes solo backups en el mismo disco del servidor.
Cifra backups si contienen datos sensibles.
Monitorea que el backup termine correctamente.
```

---

# Parte IX: logs y observabilidad

## 36. Ver logs

Archivo típico:

```bash
sudo tail -f /var/log/postgresql/postgresql-18-main.log
```

Últimas líneas:

```bash
sudo tail -n 100 /var/log/postgresql/postgresql-18-main.log
```

Con journalctl:

```bash
sudo journalctl -u postgresql
```

---

## 37. Configurar logs útiles

En `postgresql.conf`:

```conf
log_min_duration_statement = 500
log_lock_waits = on
deadlock_timeout = '1s'
log_connections = off
log_disconnections = off
log_checkpoints = on
```

Significado:

```text
log_min_duration_statement = 500:
Loguea consultas que tarden más de 500 ms.

log_lock_waits = on:
Loguea esperas por locks.

deadlock_timeout = 1s:
Tiempo antes de revisar deadlocks y loguear lock waits.

log_checkpoints = on:
Ayuda a diagnosticar presión de checkpoints.
```

Recargar:

```bash
sudo pg_ctlcluster 18 main reload
```

---

## 38. Logs en producción

Buenas prácticas:

```text
Loguea consultas lentas.
No loguees todos los statements en producción salvo diagnóstico temporal.
Centraliza logs.
Define retención.
Monitorea errores.
No loguees datos sensibles innecesariamente.
```

Evita dejar esto permanentemente en producción:

```conf
log_statement = 'all'
```

Puede generar volumen alto y exponer datos sensibles.

---

## 39. Consultas de monitoreo

Conexiones activas:

```sql
SELECT
    pid,
    usename,
    application_name,
    client_addr,
    state,
    wait_event_type,
    wait_event,
    query
FROM pg_stat_activity
WHERE state <> 'idle';
```

Tamaño de bases:

```sql
SELECT
    datname,
    pg_size_pretty(pg_database_size(datname)) AS size
FROM pg_database
ORDER BY pg_database_size(datname) DESC;
```

Tamaño de tablas:

```sql
SELECT
    schemaname,
    relname,
    pg_size_pretty(pg_total_relation_size(relid)) AS total_size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC
LIMIT 20;
```

Índices no usados, revisar con cuidado:

```sql
SELECT
    schemaname,
    relname,
    indexrelname,
    idx_scan
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC
LIMIT 20;
```

---

## 40. pg_stat_statements

`pg_stat_statements` ayuda a identificar consultas pesadas.

Instalar extensión:

```bash
sudo apt install postgresql-contrib-18
```

En `postgresql.conf`:

```conf
shared_preload_libraries = 'pg_stat_statements'
```

Reiniciar:

```bash
sudo pg_ctlcluster 18 main restart
```

Crear extensión en la base:

```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```

Consultar:

```sql
SELECT
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

Advertencia:

```text
shared_preload_libraries requiere restart.
```

---

# Parte X: mantenimiento

## 41. VACUUM y ANALYZE

PostgreSQL usa MVCC, por lo que pueden quedar filas muertas después de updates/deletes.

Comandos:

```sql
VACUUM;
ANALYZE;
VACUUM ANALYZE;
```

Ver autovacuum:

```sql
SHOW autovacuum;
```

Buenas prácticas:

```text
Mantén autovacuum activado.
No uses VACUUM FULL como rutina.
Monitorea tablas con muchas filas muertas.
Usa ANALYZE para actualizar estadísticas.
```

---

## 42. Revisar filas muertas

```sql
SELECT
    schemaname,
    relname,
    n_live_tup,
    n_dead_tup,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 20;
```

---

## 43. Reindex

Usa `REINDEX` cuando un índice está corrupto o muy inflado, no por rutina diaria.

```sql
REINDEX INDEX index_name;
```

Reindexar tabla:

```sql
REINDEX TABLE table_name;
```

En producción, evalúa opciones concurrentes según versión:

```sql
REINDEX INDEX CONCURRENTLY index_name;
```

---

## 44. Actualizaciones menores

Mantén PostgreSQL actualizado con parches de seguridad y bugs.

Desde apt:

```bash
sudo apt update
sudo apt upgrade
```

Reinicia si el paquete actualizado lo requiere:

```bash
sudo systemctl restart postgresql
```

Ver versión activa:

```sql
SELECT version();
```

Ver paquetes:

```bash
apt list --installed | grep postgresql
```

---

# Parte XI: seguridad y hardening

## 45. Reglas básicas de seguridad

```text
No uses superusuario para la aplicación.
No expongas PostgreSQL a internet.
Usa contraseñas fuertes.
Usa scram-sha-256.
Usa firewall.
Usa TLS para conexiones remotas.
Limita permisos por rol.
Separa usuarios por aplicación.
Haz backups cifrados.
Monitorea logs.
Actualiza paquetes.
```

---

## 46. Crear roles con mínimo privilegio

Rol dueño de objetos:

```sql
CREATE ROLE app_owner
WITH
    LOGIN
    PASSWORD 'owner_password';
```

Rol de runtime de la aplicación:

```sql
CREATE ROLE app_runtime
WITH
    LOGIN
    PASSWORD 'runtime_password';
```

Base:

```sql
CREATE DATABASE app_db OWNER app_owner;
```

Permisos para runtime:

```sql
\c app_db

GRANT CONNECT ON DATABASE app_db TO app_runtime;
GRANT USAGE ON SCHEMA public TO app_runtime;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_runtime;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_runtime;
```

Permisos por defecto para futuras tablas:

```sql
ALTER DEFAULT PRIVILEGES FOR ROLE app_owner IN SCHEMA public
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_runtime;

ALTER DEFAULT PRIVILEGES FOR ROLE app_owner IN SCHEMA public
GRANT USAGE, SELECT ON SEQUENCES TO app_runtime;
```

Ventaja:

```text
La app no puede cambiar schema si solo usa app_runtime.
Las migraciones pueden usar app_owner.
```

---

## 47. Revocar permisos públicos innecesarios

En bases sensibles:

```sql
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
REVOKE ALL ON DATABASE app_db FROM PUBLIC;
```

Luego otorga explícitamente:

```sql
GRANT CONNECT ON DATABASE app_db TO app_runtime;
```

Cuidado:

```text
Prueba esto en staging antes de producción.
Algunas herramientas asumen permisos en public.
```

---

## 48. Evitar trust

No uses `trust` salvo casos muy controlados de desarrollo local.

Mala práctica en servidor:

```text
host    all    all    0.0.0.0/0    trust
```

`trust` permite conectar sin contraseña si la regla coincide.

Mejor:

```text
hostssl app_db app_runtime 10.0.10.0/24 scram-sha-256
```

---

## 49. Proteger archivos de configuración

Revisar permisos:

```bash
ls -l /etc/postgresql/18/main/
ls -ld /var/lib/postgresql/18/main/
```

Los datos deben pertenecer a `postgres`:

```bash
sudo chown -R postgres:postgres /var/lib/postgresql/18/main
```

No edites archivos de datos manualmente.

---

# Parte XII: tuning inicial

## 50. Parámetros principales

Parámetros comunes:

```text
shared_buffers:
Memoria compartida usada por PostgreSQL.

effective_cache_size:
Estimación de cache disponible del sistema operativo.

work_mem:
Memoria por operación de sort/hash, por conexión.

maintenance_work_mem:
Memoria para vacuum, create index, alter table.

max_connections:
Máximo de conexiones concurrentes.

checkpoint_timeout:
Frecuencia máxima entre checkpoints.

max_wal_size:
Tamaño WAL antes de forzar checkpoint.

random_page_cost:
Costo relativo de lecturas aleatorias para el planner.
```

---

## 51. Valores iniciales orientativos

Para desarrollo local:

```conf
shared_buffers = 256MB
effective_cache_size = 1GB
work_mem = 16MB
maintenance_work_mem = 128MB
```

Para servidor pequeño con 4 GB RAM dedicado parcialmente:

```conf
shared_buffers = 1GB
effective_cache_size = 3GB
work_mem = 16MB
maintenance_work_mem = 256MB
max_connections = 100
```

Para servidor dedicado de 16 GB RAM:

```conf
shared_buffers = 4GB
effective_cache_size = 12GB
work_mem = 32MB
maintenance_work_mem = 1GB
max_connections = 100
```

Advertencia:

```text
work_mem puede multiplicarse por conexión y por operación.
No lo subas sin entender la carga.
```

Regla:

```text
Mide antes y después.
Cambia pocos parámetros a la vez.
Documenta cada cambio.
```

---

## 52. Conexiones y pooling

Si tu app abre muchas conexiones, usa pooling.

Opciones:

```text
Pool interno de SQLAlchemy/psycopg.
PgBouncer.
Pool de framework.
RDS Proxy si estás en AWS.
```

Problemas de demasiadas conexiones:

```text
Memoria alta.
max_connections agotado.
Latencia.
Contención.
Errores intermitentes.
```

Consulta conexiones:

```sql
SELECT
    datname,
    usename,
    state,
    count(*)
FROM pg_stat_activity
GROUP BY datname, usename, state
ORDER BY count(*) DESC;
```

---

# Parte XIII: extensiones útiles

## 53. Instalar contrib

Si no lo instalaste:

```bash
sudo apt install postgresql-contrib-18
```

Crear extensiones:

```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

Extensiones frecuentes:

```text
pg_stat_statements:
Estadísticas de consultas.

pgcrypto:
Funciones criptográficas, gen_random_uuid().

uuid-ossp:
Generación de UUIDs, más histórica.

citext:
Texto case-insensitive.

pg_trgm:
Búsqueda por similitud.

postgis:
Datos geoespaciales, paquete separado.
```

PostGIS:

```bash
sudo apt install postgresql-18-postgis-3
```

Crear:

```sql
CREATE EXTENSION IF NOT EXISTS postgis;
```

---

# Parte XIV: múltiples versiones y clusters

## 54. Ver clusters instalados

```bash
pg_lsclusters
```

Ejemplo:

```text
Ver Cluster Port Status Owner    Data directory              Log file
16  main    5432 online postgres /var/lib/postgresql/16/main ...
18  main    5433 online postgres /var/lib/postgresql/18/main ...
```

Cada cluster puede usar un puerto distinto.

---

## 55. Crear un cluster nuevo

```bash
sudo pg_createcluster 18 testcluster --port 5433
```

Iniciar:

```bash
sudo pg_ctlcluster 18 testcluster start
```

Conectar:

```bash
sudo -u postgres psql -p 5433
```

Eliminar cluster, cuidado, borra datos:

```bash
sudo pg_dropcluster 18 testcluster --stop
```

---

## 56. Cambiar puerto de un cluster

Editar:

```bash
sudo nano /etc/postgresql/18/main/postgresql.conf
```

Cambiar:

```conf
port = 5433
```

Reiniciar:

```bash
sudo pg_ctlcluster 18 main restart
```

Ver:

```bash
pg_lsclusters
```

---

# Parte XV: upgrades

## 57. Actualizaciones menores vs mayores

```text
Minor upgrade:
18.3 -> 18.4
Normalmente no cambia formato de datos.
Se instala con apt upgrade.

Major upgrade:
17 -> 18
Requiere migración del cluster.
Puede requerir pg_upgrade, dump/restore o herramienta gestionada.
```

---

## 58. Actualización menor

```bash
sudo apt update
sudo apt upgrade
```

Reiniciar si es necesario:

```bash
sudo systemctl restart postgresql
```

Ver:

```sql
SELECT version();
```

---

## 59. Upgrade mayor con dump/restore

Método simple pero con más downtime:

```bash
pg_dumpall -U postgres > full_backup.sql
```

Instalar nueva versión:

```bash
sudo apt install postgresql-18 postgresql-client-18
```

Restaurar en nuevo cluster:

```bash
psql -U postgres -p 5433 -f full_backup.sql
```

Ventajas:

```text
Simple de entender.
Limpio.
Útil para bases pequeñas.
```

Desventajas:

```text
Más lento.
Más downtime.
Requiere espacio adicional.
```

---

## 60. Upgrade mayor con pg_upgrade

`pg_upgrade` migra datos entre versiones mayores con menos tiempo que dump/restore en muchos casos.

Flujo conceptual:

```text
1. Hacer backup.
2. Instalar nueva versión.
3. Detener clusters.
4. Ejecutar pg_upgrade.
5. Analizar nueva base.
6. Probar.
7. Eliminar cluster viejo solo cuando estés seguro.
```

En Ubuntu/Debian, existe `pg_upgradecluster`, que simplifica upgrades entre clusters empaquetados.

Ejemplo conceptual:

```bash
sudo pg_upgradecluster 16 main
```

Advertencia:

```text
Prueba upgrades en staging.
Lee release notes.
Verifica extensiones.
Ten rollback.
Haz backup antes.
```

---

# Parte XVI: integración con Python

## 61. Instalar cliente Python

Opción moderna con psycopg 3:

```bash
pip install "psycopg[binary]"
```

Para producción, algunos equipos prefieren compilar contra libpq:

```bash
sudo apt install libpq-dev python3-dev build-essential
pip install psycopg
```

SQLAlchemy:

```bash
pip install sqlalchemy psycopg
```

---

## 62. Conexión con psycopg

```python
import psycopg

conninfo = "postgresql://app_user:password@localhost:5432/app_db"

with psycopg.connect(conninfo) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT now()")
        print(cur.fetchone())
```

Con variables de entorno:

```python
import os
import psycopg

database_url = os.environ["DATABASE_URL"]

with psycopg.connect(database_url) as conn:
    ...
```

Regla:

```text
No hardcodees credenciales.
Usa variables de entorno o gestor de secretos.
```

---

## 63. SQLAlchemy básico

```python
from sqlalchemy import create_engine, text

engine = create_engine(
    "postgresql+psycopg://app_user:password@localhost:5432/app_db",
    pool_size=5,
    max_overflow=10,
    pool_pre_ping=True,
)

with engine.connect() as connection:
    result = connection.execute(text("SELECT now()"))
    print(result.scalar_one())
```

Buenas prácticas:

```text
Usa pool_pre_ping para conexiones largas.
Configura pool_size según capacidad de PostgreSQL.
No abras una conexión nueva por request sin pooling.
```

---

# Parte XVII: troubleshooting

## 64. Error: peer authentication failed

Mensaje típico:

```text
FATAL: Peer authentication failed for user "app_user"
```

Causa:

```text
Estás conectando por socket local y pg_hba.conf usa peer.
```

Soluciones:

```text
Usar -h localhost para TCP con contraseña.
Crear usuario Linux del mismo nombre.
Cambiar pg_hba.conf para ese caso.
```

Ejemplo:

```bash
psql -h localhost -U app_user -d app_db
```

---

## 65. Error: connection refused

Posibles causas:

```text
PostgreSQL no está corriendo.
Puerto incorrecto.
listen_addresses no incluye esa IP.
Firewall bloquea.
Servicio escucha solo en localhost.
```

Diagnóstico:

```bash
sudo systemctl status postgresql
pg_lsclusters
ss -ltnp | grep 5432
pg_isready -h localhost -p 5432
```

---

## 66. Error: no pg_hba.conf entry

Mensaje:

```text
FATAL: no pg_hba.conf entry for host ...
```

Causa:

```text
PostgreSQL recibió la conexión,
pero pg_hba.conf no tiene una regla que la permita.
```

Solución:

```text
Agregar regla específica.
Recargar configuración.
Verificar pg_hba_file_rules.
```

Ejemplo:

```text
hostssl app_db app_user 10.0.10.25/32 scram-sha-256
```

Recargar:

```bash
sudo pg_ctlcluster 18 main reload
```

---

## 67. Error: password authentication failed

Causas:

```text
Contraseña incorrecta.
Usuario incorrecto.
Conexión a base incorrecta.
Password antiguo almacenado con método incompatible.
```

Solución:

```sql
ALTER ROLE app_user WITH PASSWORD 'new_secure_password';
```

Verifica `pg_hba.conf`.

---

## 68. Error: role does not exist

Mensaje:

```text
FATAL: role "myuser" does not exist
```

Crear rol:

```sql
CREATE ROLE myuser WITH LOGIN PASSWORD 'password';
```

Ver roles:

```sql
\du
```

---

## 69. Error: database does not exist

Crear base:

```sql
CREATE DATABASE app_db OWNER app_user;
```

Ver bases:

```sql
\l
```

---

## 70. Disco lleno

Diagnóstico:

```bash
df -h
sudo du -sh /var/lib/postgresql/*
sudo du -sh /var/log/postgresql/*
```

Desde SQL:

```sql
SELECT
    datname,
    pg_size_pretty(pg_database_size(datname))
FROM pg_database;
```

Acciones:

```text
No borres archivos de /var/lib/postgresql manualmente.
Revisa logs.
Revisa backups locales.
Revisa tablas grandes.
Revisa WAL si hay replicación/archivado roto.
Agrega almacenamiento si es necesario.
```

---

# Parte XVIII: desinstalación

## 71. Desinstalar paquetes

Detener:

```bash
sudo systemctl stop postgresql
```

Eliminar paquetes:

```bash
sudo apt remove postgresql postgresql-contrib
```

Eliminar versión específica:

```bash
sudo apt remove postgresql-18 postgresql-client-18 postgresql-contrib-18
```

Eliminar configuración también:

```bash
sudo apt purge postgresql-18 postgresql-client-18 postgresql-contrib-18
```

Cuidado:

```text
Esto puede eliminar configuración.
No elimina necesariamente todos los datos si quedan clusters.
Haz backup antes.
```

---

## 72. Eliminar cluster

Ver clusters:

```bash
pg_lsclusters
```

Eliminar cluster específico:

```bash
sudo pg_dropcluster 18 main --stop
```

Advertencia:

```text
Esto elimina datos del cluster.
Haz backup antes.
```

---

# Parte XIX: checklist final

## 73. Checklist de instalación local

```text
[ ] apt update ejecutado.
[ ] PostgreSQL instalado.
[ ] Servicio activo.
[ ] pg_lsclusters muestra cluster online.
[ ] psql funciona.
[ ] Usuario app creado.
[ ] Base app creada.
[ ] Conexión local probada.
```

---

## 74. Checklist de servidor

```text
[ ] No se usa usuario postgres desde la app.
[ ] app_user tiene permisos mínimos.
[ ] password_encryption = scram-sha-256.
[ ] pg_hba.conf no usa trust.
[ ] listen_addresses no está abierto sin necesidad.
[ ] UFW/firewall restringe puerto 5432.
[ ] TLS configurado si hay acceso remoto.
[ ] Logs de consultas lentas activos.
[ ] Backups automatizados.
[ ] Restauración probada.
[ ] Actualizaciones menores planificadas.
```

---

## 75. Checklist de producción pequeña/mediana

```text
[ ] PostgreSQL no expuesto a internet.
[ ] Acceso por red privada, VPN o túnel.
[ ] Backups fuera del servidor.
[ ] Backups cifrados si hay datos sensibles.
[ ] Monitoreo de CPU, RAM, disco y conexiones.
[ ] Alertas de disco bajo.
[ ] Alertas de servicio caído.
[ ] pg_stat_statements instalado.
[ ] Pooling configurado en aplicación.
[ ] Migraciones versionadas.
[ ] Runbook de restauración documentado.
[ ] Prueba de restore periódica.
```

---

# Parte XX: comandos rápidos

## 76. Resumen de comandos

Instalar desde Ubuntu:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

Instalar desde PGDG:

```bash
sudo apt install -y postgresql-common ca-certificates
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
sudo apt update
sudo apt install postgresql-18 postgresql-client-18 postgresql-contrib-18
```

Estado:

```bash
sudo systemctl status postgresql
pg_lsclusters
pg_isready
```

Entrar:

```bash
sudo -u postgres psql
```

Crear usuario y base:

```sql
CREATE ROLE app_user WITH LOGIN PASSWORD 'change_this_password';
CREATE DATABASE app_db OWNER app_user;
```

Conectar:

```bash
psql -h localhost -U app_user -d app_db
```

Recargar:

```bash
sudo pg_ctlcluster 18 main reload
```

Reiniciar:

```bash
sudo pg_ctlcluster 18 main restart
```

Backup:

```bash
pg_dump -U app_user -h localhost -d app_db -Fc -f app_db.dump
```

Restore:

```bash
createdb -U postgres -h localhost app_db_restore
pg_restore -U postgres -h localhost -d app_db_restore app_db.dump
```

Logs:

```bash
sudo tail -f /var/log/postgresql/postgresql-18-main.log
```

---

# Fuentes de referencia recomendadas

```text
- PostgreSQL: Linux downloads for Ubuntu.
- PostgreSQL Apt Repository, PGDG.
- Ubuntu Server documentation: Install and configure PostgreSQL.
- PostgreSQL documentation: Client authentication and pg_hba.conf.
- PostgreSQL documentation: Connection settings and listen_addresses.
- PostgreSQL documentation: pg_dump, pg_restore and pg_isready.
- Ubuntu manpages: pg_ctlcluster.
```
