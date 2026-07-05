# Guía de instalación y configuración de Apache en Ubuntu

## Objetivo

Esta guía explica cómo instalar, configurar, asegurar y operar Apache HTTP Server en Ubuntu. Está orientada a servidores web reales: sitios estáticos, múltiples dominios con Virtual Hosts, HTTPS con Let’s Encrypt, reverse proxy para aplicaciones backend, logs, firewall, módulos, rendimiento y troubleshooting.

Está organizada en tres niveles:

```text
1. Básico: instalación, servicio, archivos importantes, sitio por defecto, firewall y comandos esenciales.
2. Intermedio: Virtual Hosts, HTTPS, módulos, logs, redirecciones, compresión, headers y reverse proxy.
3. Avanzado: hardening, performance, MPM, PHP-FPM, balanceo, monitoreo, despliegue, backups de configuración y troubleshooting.
```

La guía usa Ubuntu Server moderno y Apache 2.4, que es la línea usada actualmente en Ubuntu.

---

# Parte I: conceptos básicos

## 1. Qué es Apache

Apache HTTP Server, normalmente llamado Apache o Apache2 en Ubuntu, es un servidor web. Puede servir:

```text
Sitios HTML/CSS/JS estáticos.
Aplicaciones PHP.
APIs detrás de reverse proxy.
Aplicaciones Python, Node.js, Java, Go u otros backends.
Contenido protegido por autenticación.
Múltiples dominios en el mismo servidor.
HTTPS con certificados TLS.
```

Apache puede actuar como:

```text
Servidor web directo.
Reverse proxy.
Terminador TLS.
Servidor de archivos estáticos.
Balanceador simple.
Punto de entrada para aplicaciones internas.
```

---

## 2. Apache en Ubuntu

En Ubuntu, el paquete se llama:

```text
apache2
```

Comando de instalación:

```bash
sudo apt update
sudo apt install apache2
```

Estructura típica en Ubuntu:

```text
/etc/apache2/
    apache2.conf
    ports.conf
    envvars
    mods-available/
    mods-enabled/
    sites-available/
    sites-enabled/
    conf-available/
    conf-enabled/

/var/www/html/
    index.html

/var/log/apache2/
    access.log
    error.log
```

Ubuntu usa helpers como:

```text
a2ensite      habilitar sitio
a2dissite     deshabilitar sitio
a2enmod       habilitar módulo
a2dismod      deshabilitar módulo
a2enconf      habilitar configuración
a2disconf     deshabilitar configuración
apache2ctl    validar y controlar configuración
```

---

## 3. Modelo de configuración de Ubuntu

Apache en Ubuntu separa configuración en carpetas:

```text
mods-available:
Módulos disponibles.

mods-enabled:
Módulos habilitados mediante symlinks.

sites-available:
Virtual Hosts disponibles.

sites-enabled:
Virtual Hosts habilitados.

conf-available:
Fragmentos de configuración disponibles.

conf-enabled:
Fragmentos habilitados.
```

Regla:

```text
No edites directamente sites-enabled.
Crea o edita archivos en sites-available y habilítalos con a2ensite.
```

Ejemplo:

```bash
sudo nano /etc/apache2/sites-available/example.com.conf
sudo a2ensite example.com.conf
sudo systemctl reload apache2
```

---

## 4. Puertos comunes

Apache suele usar:

```text
80/tcp   HTTP
443/tcp  HTTPS
```

Archivo:

```bash
/etc/apache2/ports.conf
```

Ejemplo:

```apache
Listen 80

<IfModule ssl_module>
    Listen 443
</IfModule>
```

---

## 5. Cuándo usar Apache

Apache es buena opción cuando necesitas:

```text
Virtual Hosts simples y maduros.
Compatibilidad amplia.
.htaccess en hosting tradicional.
PHP con mod_php o PHP-FPM.
Reverse proxy estable.
TLS con mod_ssl.
Reglas por directorio.
Control fino por módulos.
```

Para aplicaciones modernas, Apache suele usarse como:

```text
Internet -> Apache HTTPS -> reverse proxy -> aplicación interna
```

Ejemplo:

```text
Apache escucha en 443.
FastAPI escucha internamente en 127.0.0.1:8000.
Apache reenvía /api hacia FastAPI.
```

---

# Parte II: instalación básica

## 6. Actualizar paquetes

Antes de instalar:

```bash
sudo apt update
sudo apt upgrade
```

Opcionalmente reinicia si hubo actualizaciones de kernel o sistema:

```bash
sudo reboot
```

---

## 7. Instalar Apache

```bash
sudo apt install apache2
```

Verificar versión:

```bash
apache2 -v
```

Verificar servicio:

```bash
sudo systemctl status apache2
```

Comprobar respuesta local:

```bash
curl -I http://localhost
```

Salida esperada:

```text
HTTP/1.1 200 OK
Server: Apache/2.4...
```

---

## 8. Probar desde navegador

Si el servidor tiene IP pública o privada accesible:

```text
http://IP_DEL_SERVIDOR
```

Deberías ver la página por defecto de Apache en Ubuntu.

Ruta del archivo:

```bash
/var/www/html/index.html
```

---

## 9. Comandos de servicio

Iniciar:

```bash
sudo systemctl start apache2
```

Detener:

```bash
sudo systemctl stop apache2
```

Reiniciar:

```bash
sudo systemctl restart apache2
```

Recargar configuración sin cortar conexiones activas:

```bash
sudo systemctl reload apache2
```

Ver estado:

```bash
sudo systemctl status apache2
```

Habilitar al arrancar:

```bash
sudo systemctl enable apache2
```

Deshabilitar al arrancar:

```bash
sudo systemctl disable apache2
```

---

## 10. Validar configuración

Antes de recargar o reiniciar:

```bash
sudo apache2ctl configtest
```

Salida correcta:

```text
Syntax OK
```

También puedes usar:

```bash
sudo apachectl -t
```

Regla:

```text
Siempre ejecuta configtest antes de reload/restart en servidores importantes.
```

---

## 11. Firewall con UFW

Ver aplicaciones disponibles:

```bash
sudo ufw app list
```

Suelen aparecer perfiles:

```text
Apache
Apache Full
Apache Secure
OpenSSH
```

Significado:

```text
Apache:
Permite puerto 80.

Apache Secure:
Permite puerto 443.

Apache Full:
Permite 80 y 443.
```

Permitir Apache completo:

```bash
sudo ufw allow "Apache Full"
```

Permitir SSH si estás conectado remotamente:

```bash
sudo ufw allow OpenSSH
```

Activar UFW:

```bash
sudo ufw enable
```

Ver estado:

```bash
sudo ufw status verbose
```

Regla:

```text
Antes de activar UFW en un servidor remoto, asegúrate de permitir OpenSSH.
```

---

## 12. Crear una página simple

Crear directorio:

```bash
sudo mkdir -p /var/www/example.com/public_html
```

Asignar propietario:

```bash
sudo chown -R "$USER":"$USER" /var/www/example.com/public_html
```

Crear archivo:

```bash
nano /var/www/example.com/public_html/index.html
```

Contenido:

```html
<!doctype html>
<html lang="es">
<head>
    <meta charset="utf-8">
    <title>example.com</title>
</head>
<body>
    <h1>Apache funcionando</h1>
    <p>Sitio servido desde Ubuntu.</p>
</body>
</html>
```

Permisos recomendados:

```bash
sudo find /var/www/example.com -type d -exec chmod 755 {} \;
sudo find /var/www/example.com -type f -exec chmod 644 {} \;
```

---

# Parte III: Virtual Hosts

## 13. Qué es un Virtual Host

Un Virtual Host permite servir varios sitios desde el mismo Apache.

Ejemplo:

```text
example.com
api.example.com
admin.example.com
```

Todos pueden vivir en el mismo servidor, pero con configuraciones distintas.

---

## 14. Crear Virtual Host básico

Crear archivo:

```bash
sudo nano /etc/apache2/sites-available/example.com.conf
```

Contenido:

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com

    ServerAdmin admin@example.com
    DocumentRoot /var/www/example.com/public_html

    ErrorLog ${APACHE_LOG_DIR}/example.com_error.log
    CustomLog ${APACHE_LOG_DIR}/example.com_access.log combined

    <Directory /var/www/example.com/public_html>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```

Habilitar sitio:

```bash
sudo a2ensite example.com.conf
```

Deshabilitar sitio por defecto si no lo usarás:

```bash
sudo a2dissite 000-default.conf
```

Validar:

```bash
sudo apache2ctl configtest
```

Recargar:

```bash
sudo systemctl reload apache2
```

---

## 15. Probar dominio localmente

Si aún no tienes DNS, puedes editar `/etc/hosts` en tu máquina cliente:

```text
IP_DEL_SERVIDOR example.com www.example.com
```

Luego probar:

```bash
curl -I http://example.com
```

---

## 16. Configurar DNS

En tu proveedor DNS, crea registros:

```text
A      example.com       IP_DEL_SERVIDOR
A      www.example.com   IP_DEL_SERVIDOR
```

Si usas IPv6:

```text
AAAA   example.com       IPv6_DEL_SERVIDOR
AAAA   www.example.com   IPv6_DEL_SERVIDOR
```

Antes de pedir certificado TLS, verifica:

```bash
dig example.com
dig www.example.com
```

---

## 17. Múltiples dominios

Ejemplo:

```text
/var/www/site1.com/public_html
/var/www/site2.com/public_html
```

Virtual Host 1:

```apache
<VirtualHost *:80>
    ServerName site1.com
    DocumentRoot /var/www/site1.com/public_html
</VirtualHost>
```

Virtual Host 2:

```apache
<VirtualHost *:80>
    ServerName site2.com
    DocumentRoot /var/www/site2.com/public_html
</VirtualHost>
```

Regla:

```text
Cada dominio debería tener su propio archivo .conf en sites-available.
```

---

## 18. Orden de Virtual Hosts

Si Apache recibe una request cuyo Host no coincide con ningún Virtual Host, usa el primer Virtual Host cargado para ese puerto.

Por eso conviene tener un sitio por defecto controlado.

Ejemplo:

```bash
/etc/apache2/sites-available/000-default.conf
```

Puedes crear un default que no revele información:

```apache
<VirtualHost *:80>
    ServerName default.local

    DocumentRoot /var/www/empty

    <Directory /var/www/empty>
        Require all denied
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/default_error.log
    CustomLog ${APACHE_LOG_DIR}/default_access.log combined
</VirtualHost>
```

---

# Parte IV: HTTPS con Let’s Encrypt

## 19. Por qué usar HTTPS

HTTPS protege:

```text
Credenciales.
Cookies.
Tokens.
Formularios.
Contenido privado.
Integridad de la respuesta.
Confianza del navegador.
```

En producción, todo sitio web debería usar HTTPS.

---

## 20. Instalar Certbot

En Ubuntu, una forma común es usar Snap.

Instalar Snap si no está:

```bash
sudo apt install snapd
```

Instalar Certbot:

```bash
sudo snap install --classic certbot
```

Crear symlink:

```bash
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Verificar:

```bash
certbot --version
```

---

## 21. Requisitos antes de pedir certificado

Antes de ejecutar Certbot:

```text
El dominio debe resolver a la IP del servidor.
Apache debe responder por HTTP en puerto 80.
UFW o firewall debe permitir puerto 80.
El Virtual Host debe tener ServerName correcto.
```

Verificar:

```bash
curl -I http://example.com
```

---

## 22. Obtener certificado y configurar Apache automáticamente

```bash
sudo certbot --apache -d example.com -d www.example.com
```

Certbot puede:

```text
Obtener certificado.
Editar configuración de Apache.
Crear Virtual Host HTTPS.
Configurar redirección HTTP -> HTTPS.
Configurar renovación automática.
```

Verificar HTTPS:

```bash
curl -I https://example.com
```

---

## 23. Renovación automática

Certbot instala renovación automática mediante systemd timer o mecanismo equivalente.

Probar renovación:

```bash
sudo certbot renew --dry-run
```

Ver certificados:

```bash
sudo certbot certificates
```

---

## 24. Configuración HTTPS manual

Si prefieres configurar manualmente:

Habilitar SSL:

```bash
sudo a2enmod ssl
sudo systemctl reload apache2
```

Virtual Host HTTPS:

```apache
<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com

    DocumentRoot /var/www/example.com/public_html

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem

    ErrorLog ${APACHE_LOG_DIR}/example.com_ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/example.com_ssl_access.log combined

    <Directory /var/www/example.com/public_html>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```

Redirección HTTP a HTTPS:

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com

    Redirect permanent / https://example.com/
</VirtualHost>
```

Validar y recargar:

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
```

---

## 25. HSTS

HSTS indica al navegador que use HTTPS en futuras conexiones.

Configurar:

```apache
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
```

Requiere módulo headers:

```bash
sudo a2enmod headers
sudo systemctl reload apache2
```

Advertencia:

```text
Activa HSTS solo cuando estés seguro de que HTTPS funciona correctamente en el dominio y subdominios incluidos.
```

Para preload:

```apache
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
```

No uses `preload` sin entender sus consecuencias.

---

# Parte V: módulos de Apache

## 26. Ver módulos habilitados

```bash
apache2ctl -M
```

Listar módulos disponibles:

```bash
ls /etc/apache2/mods-available
```

Habilitar módulo:

```bash
sudo a2enmod rewrite
sudo systemctl reload apache2
```

Deshabilitar:

```bash
sudo a2dismod rewrite
sudo systemctl reload apache2
```

---

## 27. Módulos comunes

| Módulo | Uso |
|---|---|
| ssl | HTTPS/TLS |
| rewrite | Reglas de reescritura |
| headers | Headers HTTP |
| proxy | Reverse proxy base |
| proxy_http | Reverse proxy HTTP |
| proxy_wstunnel | WebSockets |
| http2 | HTTP/2 |
| deflate | Compresión gzip |
| brotli | Compresión Brotli si está disponible |
| expires | Cache headers |
| status | Estado interno |
| remoteip | IP real detrás de proxy/CDN |
| security2 | ModSecurity, WAF opcional |

---

## 28. Habilitar módulos frecuentes

Para HTTPS:

```bash
sudo a2enmod ssl headers
```

Para reverse proxy:

```bash
sudo a2enmod proxy proxy_http
```

Para WebSockets:

```bash
sudo a2enmod proxy_wstunnel
```

Para reescrituras:

```bash
sudo a2enmod rewrite
```

Para compresión:

```bash
sudo a2enmod deflate
```

Para HTTP/2:

```bash
sudo a2enmod http2
```

Aplicar:

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
```

---

# Parte VI: reverse proxy para aplicaciones web

## 29. Qué es un reverse proxy

Un reverse proxy recibe requests públicas y las reenvía a una aplicación interna.

Ejemplo:

```text
Usuario -> Apache :443 -> FastAPI :8000 en localhost
```

Ventajas:

```text
Apache maneja HTTPS.
La app no se expone directamente.
Apache puede servir estáticos.
Apache puede aplicar headers y logs.
Permite varios backends bajo un mismo dominio.
```

---

## 30. Reverse proxy simple

Supongamos que tu app corre en:

```text
127.0.0.1:8000
```

Habilitar módulos:

```bash
sudo a2enmod proxy proxy_http headers
sudo systemctl reload apache2
```

Virtual Host:

```apache
<VirtualHost *:80>
    ServerName api.example.com

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/

    ErrorLog ${APACHE_LOG_DIR}/api.example.com_error.log
    CustomLog ${APACHE_LOG_DIR}/api.example.com_access.log combined
</VirtualHost>
```

Con HTTPS:

```apache
<VirtualHost *:443>
    ServerName api.example.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/api.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/api.example.com/privkey.pem

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/

    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Port "443"

    ErrorLog ${APACHE_LOG_DIR}/api.example.com_ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/api.example.com_ssl_access.log combined
</VirtualHost>
```

---

## 31. Reverse proxy para FastAPI

FastAPI con Uvicorn:

```bash
uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Apache:

```apache
<VirtualHost *:443>
    ServerName api.example.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/api.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/api.example.com/privkey.pem

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/

    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Host "api.example.com"

    ErrorLog ${APACHE_LOG_DIR}/api_error.log
    CustomLog ${APACHE_LOG_DIR}/api_access.log combined
</VirtualHost>
```

En FastAPI/Uvicorn, si dependes de headers proxy, configura correctamente trusted hosts/proxy headers según tu despliegue.

---

## 32. Reverse proxy por path

Ejemplo:

```text
example.com/      sitio estático
example.com/api/  backend interno
```

Virtual Host:

```apache
<VirtualHost *:443>
    ServerName example.com
    DocumentRoot /var/www/example.com/public_html

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem

    ProxyPreserveHost On
    ProxyPass /api/ http://127.0.0.1:8000/
    ProxyPassReverse /api/ http://127.0.0.1:8000/

    <Directory /var/www/example.com/public_html>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```

Cuidado:

```text
El backend debe saber si vive bajo /api/ o si Apache elimina ese prefijo.
Prueba URLs generadas, redirects y documentación OpenAPI.
```

---

## 33. WebSockets

Habilitar:

```bash
sudo a2enmod proxy proxy_http proxy_wstunnel
sudo systemctl reload apache2
```

Config:

```apache
ProxyPass /ws/ ws://127.0.0.1:8000/ws/
ProxyPassReverse /ws/ ws://127.0.0.1:8000/ws/
```

Ejemplo completo:

```apache
<VirtualHost *:443>
    ServerName realtime.example.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/realtime.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/realtime.example.com/privkey.pem

    ProxyPreserveHost On

    ProxyPass /ws/ ws://127.0.0.1:8000/ws/
    ProxyPassReverse /ws/ ws://127.0.0.1:8000/ws/

    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/
</VirtualHost>
```

---

## 34. No habilitar proxy abierto

No uses esto sin entenderlo:

```apache
ProxyRequests On
```

Para reverse proxy normal, debe estar desactivado:

```apache
ProxyRequests Off
```

Regla:

```text
Un proxy abierto puede ser abusado por terceros.
Para reverse proxy usa ProxyPass, no ProxyRequests On.
```

---

# Parte VII: PHP con Apache

## 35. Opciones para PHP

Opciones comunes:

```text
mod_php:
PHP embebido en Apache.

PHP-FPM:
PHP corre como servicio separado; Apache se comunica por FastCGI.
```

Recomendación actual:

```text
Usa PHP-FPM para producción.
```

---

## 36. Instalar PHP-FPM

```bash
sudo apt install php-fpm
```

Ver versión:

```bash
php -v
```

Ver servicio:

```bash
systemctl status php*-fpm
```

Habilitar módulos Apache:

```bash
sudo a2enmod proxy_fcgi setenvif
sudo a2enconf php*-fpm
sudo systemctl reload apache2
```

Crear archivo PHP:

```bash
echo "<?php phpinfo();" | sudo tee /var/www/example.com/public_html/info.php
```

Probar:

```text
https://example.com/info.php
```

Después de probar, elimina `info.php`:

```bash
sudo rm /var/www/example.com/public_html/info.php
```

---

## 37. Virtual Host para PHP

```apache
<VirtualHost *:443>
    ServerName php.example.com
    DocumentRoot /var/www/php.example.com/public

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/php.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/php.example.com/privkey.pem

    <Directory /var/www/php.example.com/public>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/php.example.com_error.log
    CustomLog ${APACHE_LOG_DIR}/php.example.com_access.log combined
</VirtualHost>
```

Para frameworks PHP, muchas veces `AllowOverride All` se usa para `.htaccess`. Si puedes mover reglas a la config de Apache y usar `AllowOverride None`, suele ser más eficiente y controlado.

---

# Parte VIII: logs

## 38. Logs principales

Ubicación:

```bash
/var/log/apache2/
```

Archivos comunes:

```text
access.log
error.log
other_vhosts_access.log
example.com_access.log
example.com_error.log
```

Ver en vivo:

```bash
sudo tail -f /var/log/apache2/error.log
```

Ver últimos accesos:

```bash
sudo tail -n 100 /var/log/apache2/access.log
```

---

## 39. CustomLog y ErrorLog

Por Virtual Host:

```apache
ErrorLog ${APACHE_LOG_DIR}/example.com_error.log
CustomLog ${APACHE_LOG_DIR}/example.com_access.log combined
```

Formato `combined` incluye:

```text
IP.
Identidad.
Usuario.
Fecha.
Request.
Status.
Bytes.
Referer.
User-Agent.
```

---

## 40. LogLevel

Configurar nivel:

```apache
LogLevel warn
```

Niveles comunes:

```text
debug
info
notice
warn
error
crit
alert
emerg
```

En producción:

```text
warn o error suele ser razonable.
debug solo temporalmente.
```

Ejemplo por módulo:

```apache
LogLevel warn proxy:info rewrite:trace2
```

No dejes `rewrite:trace8` en producción salvo diagnóstico muy puntual.

---

## 41. Logrotate

Ubuntu rota logs de Apache automáticamente.

Archivo:

```bash
/etc/logrotate.d/apache2
```

Ver configuración:

```bash
cat /etc/logrotate.d/apache2
```

Forzar prueba, con cuidado:

```bash
sudo logrotate -d /etc/logrotate.d/apache2
```

La opción `-d` depura sin aplicar cambios.

---

# Parte IX: redirecciones y reescrituras

## 42. Redirección HTTP a HTTPS

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com

    Redirect permanent / https://example.com/
</VirtualHost>
```

O con rewrite:

```apache
RewriteEngine On
RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

Para redirección simple, `Redirect` es más claro.

---

## 43. Redirigir www a no-www

```apache
<VirtualHost *:80>
    ServerName www.example.com
    Redirect permanent / https://example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName www.example.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem

    Redirect permanent / https://example.com/
</VirtualHost>
```

---

## 44. Rewrites con mod_rewrite

Habilitar:

```bash
sudo a2enmod rewrite
sudo systemctl reload apache2
```

Ejemplo para SPA:

```apache
<Directory /var/www/app.example.com/dist>
    Options -Indexes +FollowSymLinks
    AllowOverride None
    Require all granted

    RewriteEngine On
    RewriteBase /
    RewriteRule ^index\.html$ - [L]
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule . /index.html [L]
</Directory>
```

Esto permite rutas como:

```text
/app
/tasks/123
/settings/profile
```

sin que Apache busque archivos físicos con esos nombres.

---

# Parte X: headers, compresión y cache

## 45. Headers de seguridad

Habilitar módulo:

```bash
sudo a2enmod headers
sudo systemctl reload apache2
```

Configuración:

```apache
Header always set X-Content-Type-Options "nosniff"
Header always set Referrer-Policy "strict-origin-when-cross-origin"
Header always set X-Frame-Options "SAMEORIGIN"
Header always set Permissions-Policy "geolocation=(), microphone=(), camera=()"
```

CSP básica:

```apache
Header always set Content-Security-Policy "default-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'self'"
```

Advertencia:

```text
Content-Security-Policy puede romper scripts, estilos o proveedores externos si no se configura según tu aplicación.
Prueba primero en staging.
```

---

## 46. Compresión con deflate

Habilitar:

```bash
sudo a2enmod deflate
sudo systemctl reload apache2
```

Config:

```apache
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css
    AddOutputFilterByType DEFLATE application/javascript application/json
    AddOutputFilterByType DEFLATE image/svg+xml
</IfModule>
```

No comprimas:

```text
Imágenes JPEG/PNG/WebP ya comprimidas.
ZIP.
PDF normalmente ya comprimidos.
```

---

## 47. Brotli

Si está disponible:

```bash
sudo apt install brotli
sudo a2enmod brotli
sudo systemctl reload apache2
```

Config:

```apache
<IfModule mod_brotli.c>
    AddOutputFilterByType BROTLI_COMPRESS text/html text/plain text/css
    AddOutputFilterByType BROTLI_COMPRESS application/javascript application/json
    AddOutputFilterByType BROTLI_COMPRESS image/svg+xml
</IfModule>
```

Brotli suele ser útil para assets estáticos, especialmente JS/CSS.

---

## 48. Cache de assets estáticos

Habilitar:

```bash
sudo a2enmod expires headers
sudo systemctl reload apache2
```

Config:

```apache
<IfModule mod_expires.c>
    ExpiresActive On

    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType image/jpeg "access plus 1 year"
    ExpiresByType image/webp "access plus 1 year"
    ExpiresByType image/svg+xml "access plus 1 year"
</IfModule>
```

Para assets con hash en nombre:

```apache
<FilesMatch "\.(css|js|png|jpg|jpeg|webp|svg|woff2)$">
    Header set Cache-Control "public, max-age=31536000, immutable"
</FilesMatch>
```

No apliques cache largo a `index.html` de una SPA:

```apache
<FilesMatch "^index\.html$">
    Header set Cache-Control "no-cache"
</FilesMatch>
```

---

# Parte XI: control de acceso

## 49. Bloquear listado de directorios

Evita:

```apache
Options Indexes
```

Usa:

```apache
Options -Indexes +FollowSymLinks
```

Ejemplo:

```apache
<Directory /var/www/example.com/public_html>
    Options -Indexes +FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

---

## 50. Restringir por IP

Ejemplo para panel interno:

```apache
<Location /admin>
    Require ip 10.0.0.0/8
    Require ip 192.168.1.0/24
</Location>
```

Para una IP concreta:

```apache
<Location /admin>
    Require ip 203.0.113.10
</Location>
```

---

## 51. Basic Auth

Instalar herramienta:

```bash
sudo apt install apache2-utils
```

Crear archivo de passwords:

```bash
sudo htpasswd -c /etc/apache2/.htpasswd admin
```

Agregar usuario adicional:

```bash
sudo htpasswd /etc/apache2/.htpasswd another_user
```

Config:

```apache
<Location /private>
    AuthType Basic
    AuthName "Restricted"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Location>
```

Recargar:

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
```

Uso recomendado:

```text
Protección simple para staging, dashboards internos o sitios temporales.
No reemplaza un sistema serio de autenticación para aplicaciones complejas.
```

---

## 52. Denegar archivos sensibles

Bloquear dotfiles:

```apache
<FilesMatch "^\.">
    Require all denied
</FilesMatch>
```

Bloquear archivos de configuración:

```apache
<FilesMatch "\.(env|ini|log|sql|bak|backup)$">
    Require all denied
</FilesMatch>
```

Regla:

```text
No pongas .env, backups ni dumps dentro de DocumentRoot.
```

---

# Parte XII: hardening

## 53. Ocultar información de versión

Archivo:

```bash
sudo nano /etc/apache2/conf-available/security.conf
```

Config recomendada:

```apache
ServerTokens Prod
ServerSignature Off
TraceEnable Off
```

Habilitar si no lo está:

```bash
sudo a2enconf security
sudo systemctl reload apache2
```

Significado:

```text
ServerTokens Prod:
Reduce información del header Server.

ServerSignature Off:
Evita firma de versión en páginas de error.

TraceEnable Off:
Desactiva método TRACE.
```

---

## 54. Permisos de archivos

Apache suele correr como:

```text
www-data
```

Ver:

```bash
ps aux | grep apache2
```

Reglas:

```text
Archivos servidos: 644.
Directorios: 755.
No dar permisos 777.
El usuario de deploy puede escribir.
Apache solo necesita leer, salvo directorios de uploads.
```

Ejemplo:

```bash
sudo chown -R deploy:www-data /var/www/example.com
sudo find /var/www/example.com -type d -exec chmod 755 {} \;
sudo find /var/www/example.com -type f -exec chmod 644 {} \;
```

Uploads:

```bash
sudo mkdir -p /var/www/example.com/uploads
sudo chown -R www-data:www-data /var/www/example.com/uploads
sudo chmod 750 /var/www/example.com/uploads
```

---

## 55. Desactivar módulos innecesarios

Ver módulos:

```bash
apache2ctl -M
```

Deshabilitar uno:

```bash
sudo a2dismod status
sudo systemctl reload apache2
```

No deshabilites módulos sin revisar dependencias.

Regla:

```text
Menos módulos habilitados = menor superficie de ataque.
```

---

## 56. Limitar métodos HTTP

Ejemplo:

```apache
<Directory /var/www/example.com/public_html>
    <LimitExcept GET POST HEAD>
        Require all denied
    </LimitExcept>
</Directory>
```

Cuidado:

```text
APIs REST pueden necesitar PUT, PATCH, DELETE u OPTIONS.
No bloquees métodos que tu aplicación necesita.
```

---

## 57. ModSecurity

ModSecurity puede actuar como WAF.

Instalación básica:

```bash
sudo apt install libapache2-mod-security2
sudo a2enmod security2
sudo systemctl restart apache2
```

Config base:

```bash
sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
```

Editar:

```bash
sudo nano /etc/modsecurity/modsecurity.conf
```

Cambiar:

```apache
SecRuleEngine On
```

Regla:

```text
ModSecurity puede generar falsos positivos.
Actívalo primero en DetectionOnly en staging si el sistema es crítico.
```

---

## 58. Fail2ban para Apache

Instalar:

```bash
sudo apt install fail2ban
```

Crear jail local:

```bash
sudo nano /etc/fail2ban/jail.local
```

Ejemplo:

```ini
[apache-auth]
enabled = true
port = http,https
logpath = %(apache_error_log)s

[apache-badbots]
enabled = true
port = http,https
logpath = %(apache_access_log)s
```

Reiniciar:

```bash
sudo systemctl restart fail2ban
```

Ver estado:

```bash
sudo fail2ban-client status
```

---

# Parte XIII: rendimiento

## 59. MPM: prefork, worker y event

Apache usa MPMs, Multi-Processing Modules.

Comunes:

```text
prefork:
Procesos, sin threads. Tradicional con mod_php.

worker:
Procesos + threads.

event:
Similar a worker, mejor manejo de conexiones keep-alive.
```

Ver MPM activo:

```bash
apache2ctl -V | grep -i mpm
```

O:

```bash
apache2ctl -M | grep mpm
```

Para sitios modernos sin mod_php, normalmente se prefiere `mpm_event`.

---

## 60. Cambiar a mpm_event

Si usas mod_php, primero debes migrar a PHP-FPM. Luego:

```bash
sudo a2dismod mpm_prefork
sudo a2enmod mpm_event
sudo systemctl restart apache2
```

Verificar:

```bash
apache2ctl -M | grep mpm
```

---

## 61. Configurar mpm_event

Archivo típico:

```bash
sudo nano /etc/apache2/mods-available/mpm_event.conf
```

Ejemplo:

```apache
<IfModule mpm_event_module>
    StartServers             2
    MinSpareThreads          25
    MaxSpareThreads          75
    ThreadLimit              64
    ThreadsPerChild          25
    MaxRequestWorkers        150
    MaxConnectionsPerChild   0
</IfModule>
```

Conceptos:

```text
MaxRequestWorkers:
Máximo de requests concurrentes.

ThreadsPerChild:
Threads por proceso hijo.

MaxConnectionsPerChild:
Reinicia procesos después de cierto número de conexiones; 0 significa ilimitado.
```

Regla:

```text
Ajusta según RAM y carga real.
No subas MaxRequestWorkers sin medir memoria.
```

---

## 62. KeepAlive

Config:

```apache
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5
```

En sitios con tráfico alto, `KeepAliveTimeout` demasiado alto puede consumir workers.

Valor típico:

```text
2 a 5 segundos.
```

---

## 63. Timeouts

Parámetros útiles:

```apache
Timeout 60
ProxyTimeout 60
```

Para reverse proxy:

```apache
ProxyTimeout 30
```

Regla:

```text
Timeouts deben ser suficientemente largos para operaciones legítimas,
pero no tan largos que permitan consumir recursos indefinidamente.
```

---

## 64. HTTP/2

Habilitar:

```bash
sudo a2enmod http2
sudo systemctl reload apache2
```

En Virtual Host TLS:

```apache
Protocols h2 http/1.1
```

Ejemplo:

```apache
<VirtualHost *:443>
    ServerName example.com
    Protocols h2 http/1.1
    ...
</VirtualHost>
```

HTTP/2 requiere HTTPS en la práctica para navegadores modernos.

---

## 65. Benchmark básico con ab

Instalar:

```bash
sudo apt install apache2-utils
```

Probar:

```bash
ab -n 1000 -c 20 https://example.com/
```

Significado:

```text
-n 1000:
1000 requests.

-c 20:
20 concurrentes.
```

Advertencia:

```text
No hagas pruebas de carga agresivas contra producción sin ventana y monitoreo.
```

---

# Parte XIV: monitoreo

## 66. mod_status

Habilitar:

```bash
sudo a2enmod status
sudo systemctl reload apache2
```

Config segura:

```apache
<Location /server-status>
    SetHandler server-status
    Require ip 127.0.0.1
    Require ip 10.0.0.0/8
</Location>
```

Ver:

```bash
curl http://localhost/server-status
```

Con auto refresh:

```text
http://localhost/server-status?refresh=5
```

No expongas `/server-status` públicamente.

---

## 67. Métricas importantes

Monitorea:

```text
Requests por segundo.
Códigos 2xx, 3xx, 4xx, 5xx.
Latencia.
Uso de CPU.
Uso de memoria.
Workers ocupados.
Conexiones activas.
Disco disponible.
Tamaño de logs.
Errores en error.log.
Certificados próximos a vencer.
```

---

## 68. Logs para monitoreo

Comandos útiles:

Errores recientes:

```bash
sudo tail -n 100 /var/log/apache2/error.log
```

Top IPs:

```bash
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -nr | head
```

Top status codes:

```bash
awk '{print $9}' /var/log/apache2/access.log | sort | uniq -c | sort -nr
```

Requests 500:

```bash
awk '$9 ~ /^5/ {print}' /var/log/apache2/access.log | tail -n 50
```

---

# Parte XV: despliegue de sitios

## 69. Sitio estático

Estructura:

```text
/var/www/example.com/
    releases/
        20260704_120000/
            index.html
            assets/
    current -> releases/20260704_120000
```

Virtual Host:

```apache
DocumentRoot /var/www/example.com/current
```

Deploy:

```bash
sudo mkdir -p /var/www/example.com/releases/20260704_120000
sudo rsync -av dist/ /var/www/example.com/releases/20260704_120000/
sudo ln -sfn /var/www/example.com/releases/20260704_120000 /var/www/example.com/current
sudo systemctl reload apache2
```

Ventaja:

```text
Rollback rápido cambiando symlink.
```

---

## 70. Aplicación backend con systemd + Apache

App en `127.0.0.1:8000`:

```text
Apache público -> app systemd privada
```

Servicio systemd:

```ini
[Unit]
Description=My Web App
After=network.target

[Service]
User=appuser
Group=appuser
WorkingDirectory=/opt/myapp
Environment=APP_ENV=production
ExecStart=/opt/myapp/.venv/bin/uvicorn app.main:app --host 127.0.0.1 --port 8000
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Activar:

```bash
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
```

Apache reverse proxy:

```apache
ProxyPass / http://127.0.0.1:8000/
ProxyPassReverse / http://127.0.0.1:8000/
```

---

## 71. Separar estáticos y API

Ejemplo:

```text
example.com          frontend estático
api.example.com      backend
```

Ventajas:

```text
Configuración clara.
Certificados separados o SAN compartido.
Logs separados.
Escalamiento separado.
```

También puedes usar path:

```text
example.com/
example.com/api/
```

Pero requiere más cuidado con rutas y redirects del backend.

---

# Parte XVI: backups de configuración

## 72. Qué respaldar

Apache:

```text
/etc/apache2/
```

Sitios:

```text
/var/www/
```

Certificados Let’s Encrypt:

```text
/etc/letsencrypt/
```

Scripts de deploy:

```text
/opt/deploy/
```

Variables y secretos:

```text
No deben estar dispersos.
Respáldalos mediante gestor de secretos o procedimiento seguro.
```

---

## 73. Backup simple

```bash
sudo tar -czf apache_config_$(date +%Y%m%d).tar.gz /etc/apache2
```

Sitios:

```bash
sudo tar -czf www_$(date +%Y%m%d).tar.gz /var/www
```

Let’s Encrypt:

```bash
sudo tar -czf letsencrypt_$(date +%Y%m%d).tar.gz /etc/letsencrypt
```

Recomendación:

```text
Versiona configuración no secreta en Git.
No dependas solo del servidor como fuente de verdad.
```

---

# Parte XVII: troubleshooting

## 74. Apache no inicia

Ver estado:

```bash
sudo systemctl status apache2
```

Ver logs:

```bash
sudo journalctl -xeu apache2
sudo tail -n 100 /var/log/apache2/error.log
```

Validar configuración:

```bash
sudo apache2ctl configtest
```

Causas comunes:

```text
Error de sintaxis.
Puerto ocupado.
Certificado inexistente.
Ruta DocumentRoot inexistente.
Módulo requerido no habilitado.
Permisos incorrectos.
```

---

## 75. Puerto 80 o 443 ocupado

Ver procesos:

```bash
sudo ss -ltnp | grep ':80\|:443'
```

Posibles causas:

```text
Nginx instalado.
Otro Apache.
Contenedor usando puerto.
Proceso anterior colgado.
```

Detener servicio conflictivo:

```bash
sudo systemctl stop nginx
```

---

## 76. Error 403 Forbidden

Causas comunes:

```text
Permisos de archivos.
Require all granted faltante.
Options/Directory mal configurado.
DocumentRoot incorrecto.
Apache no puede atravesar directorios padre.
```

Revisar permisos:

```bash
namei -l /var/www/example.com/public_html/index.html
```

Config mínima:

```apache
<Directory /var/www/example.com/public_html>
    Options -Indexes +FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

---

## 77. Error 404 Not Found

Causas:

```text
Archivo no existe.
DocumentRoot incorrecto.
Reglas rewrite incorrectas.
Ruta de SPA no redirige a index.html.
Virtual Host equivocado.
```

Ver Virtual Hosts:

```bash
apache2ctl -S
```

---

## 78. Error 502 Bad Gateway

Común en reverse proxy.

Causas:

```text
Backend no está corriendo.
Puerto incorrecto.
ProxyPass mal configurado.
Firewall local.
Timeout.
Backend responde mal.
```

Diagnóstico:

```bash
curl -I http://127.0.0.1:8000
sudo systemctl status myapp
sudo tail -n 100 /var/log/apache2/api_error.log
```

---

## 79. Certbot falla

Causas comunes:

```text
DNS no apunta al servidor.
Puerto 80 cerrado.
Virtual Host sin ServerName correcto.
Firewall bloquea.
Cloudflare/proxy mal configurado.
Rate limit de Let’s Encrypt.
```

Verificar:

```bash
dig example.com
curl -I http://example.com
sudo ufw status
sudo apache2ctl configtest
```

Probar renovación:

```bash
sudo certbot renew --dry-run
```

---

## 80. Cambios no se aplican

Revisar:

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
apache2ctl -S
```

Posibles causas:

```text
Editaste archivo en sites-available pero no lo habilitaste.
Hay otro Virtual Host capturando el dominio.
No recargaste Apache.
Cache del navegador o CDN.
```

---

## 81. Ver configuración de Virtual Hosts

```bash
apache2ctl -S
```

Salida muestra:

```text
VirtualHost configuration.
ServerName.
Archivo y línea de configuración.
Puerto.
Default server.
```

Este comando es clave para diagnosticar dominios que caen en el Virtual Host equivocado.

---

# Parte XVIII: checklist final

## 82. Checklist de instalación básica

```text
[ ] apt update ejecutado.
[ ] apache2 instalado.
[ ] Servicio apache2 activo.
[ ] curl http://localhost responde.
[ ] UFW permite OpenSSH.
[ ] UFW permite Apache Full.
[ ] apache2ctl configtest devuelve Syntax OK.
```

---

## 83. Checklist de Virtual Host

```text
[ ] DocumentRoot existe.
[ ] index.html existe.
[ ] Permisos correctos.
[ ] ServerName correcto.
[ ] ServerAlias correcto si aplica.
[ ] Sitio habilitado con a2ensite.
[ ] Sitio por defecto deshabilitado si corresponde.
[ ] apache2ctl -S muestra el Virtual Host esperado.
[ ] Logs separados configurados.
```

---

## 84. Checklist HTTPS

```text
[ ] DNS apunta al servidor.
[ ] Puerto 80 abierto.
[ ] Certbot instalado.
[ ] Certificado emitido.
[ ] HTTP redirige a HTTPS.
[ ] Renovación dry-run exitosa.
[ ] UFW permite 443.
[ ] HSTS evaluado.
```

---

## 85. Checklist reverse proxy

```text
[ ] Backend escucha en localhost o red privada.
[ ] proxy y proxy_http habilitados.
[ ] ProxyPass correcto.
[ ] ProxyPassReverse correcto.
[ ] ProxyPreserveHost configurado si aplica.
[ ] Headers X-Forwarded revisados.
[ ] Backend no expuesto públicamente.
[ ] Health check backend funciona.
[ ] 502 diagnosticado con curl local si aparece.
```

---

## 86. Checklist seguridad

```text
[ ] ServerTokens Prod.
[ ] ServerSignature Off.
[ ] TraceEnable Off.
[ ] Options -Indexes.
[ ] No hay archivos .env dentro de DocumentRoot.
[ ] No hay backups dentro de DocumentRoot.
[ ] Permisos no usan 777.
[ ] Security headers configurados.
[ ] TLS activo.
[ ] Módulos innecesarios deshabilitados.
[ ] /server-status no es público.
[ ] Logs no exponen secretos.
```

---

## 87. Checklist producción

```text
[ ] HTTPS obligatorio.
[ ] Certificados con renovación probada.
[ ] Logs rotando.
[ ] Monitoreo de disponibilidad.
[ ] Monitoreo de 5xx.
[ ] Monitoreo de disco.
[ ] Backups de configuración.
[ ] Deploy reproducible.
[ ] Rollback definido.
[ ] Configuración versionada.
[ ] Pruebas smoke post-deploy.
```

---

# Parte XIX: comandos rápidos

## 88. Resumen de comandos

Instalar:

```bash
sudo apt update
sudo apt install apache2
```

Estado:

```bash
sudo systemctl status apache2
```

Validar:

```bash
sudo apache2ctl configtest
```

Recargar:

```bash
sudo systemctl reload apache2
```

Reiniciar:

```bash
sudo systemctl restart apache2
```

Habilitar sitio:

```bash
sudo a2ensite example.com.conf
```

Deshabilitar sitio:

```bash
sudo a2dissite example.com.conf
```

Habilitar módulo:

```bash
sudo a2enmod rewrite
```

Deshabilitar módulo:

```bash
sudo a2dismod rewrite
```

Ver módulos:

```bash
apache2ctl -M
```

Ver Virtual Hosts:

```bash
apache2ctl -S
```

Logs:

```bash
sudo tail -f /var/log/apache2/error.log
sudo tail -f /var/log/apache2/access.log
```

Firewall:

```bash
sudo ufw allow "Apache Full"
sudo ufw status verbose
```

Certbot:

```bash
sudo snap install --classic certbot
sudo certbot --apache -d example.com -d www.example.com
sudo certbot renew --dry-run
```

---

# Parte XX: configuración base recomendada

## 89. Virtual Host estático con HTTPS

```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com

    Redirect permanent / https://example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com

    DocumentRoot /var/www/example.com/public_html

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem

    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set X-Frame-Options "SAMEORIGIN"

    ErrorLog ${APACHE_LOG_DIR}/example.com_error.log
    CustomLog ${APACHE_LOG_DIR}/example.com_access.log combined

    <Directory /var/www/example.com/public_html>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    <FilesMatch "\.(css|js|png|jpg|jpeg|webp|svg|woff2)$">
        Header set Cache-Control "public, max-age=31536000, immutable"
    </FilesMatch>

    <FilesMatch "^index\.html$">
        Header set Cache-Control "no-cache"
    </FilesMatch>
</VirtualHost>
```

---

## 90. Virtual Host reverse proxy con HTTPS

```apache
<VirtualHost *:80>
    ServerName api.example.com

    Redirect permanent / https://api.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName api.example.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/api.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/api.example.com/privkey.pem

    ProxyRequests Off
    ProxyPreserveHost On

    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/

    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Port "443"

    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"

    ErrorLog ${APACHE_LOG_DIR}/api.example.com_error.log
    CustomLog ${APACHE_LOG_DIR}/api.example.com_access.log combined
</VirtualHost>
```

Módulos requeridos:

```bash
sudo a2enmod ssl headers proxy proxy_http
sudo systemctl reload apache2
```

---

# Parte XXI: resumen de reglas principales

```text
1. Instala Apache con apt install apache2.
2. Usa sites-available y a2ensite para Virtual Hosts.
3. Valida siempre con apache2ctl configtest.
4. Usa reload cuando no necesites reinicio completo.
5. Sirve cada dominio con su propio Virtual Host.
6. Usa HTTPS en producción.
7. Usa Certbot para Let’s Encrypt.
8. Deshabilita listado de directorios con Options -Indexes.
9. No pongas secretos, backups ni .env dentro del DocumentRoot.
10. Usa reverse proxy para apps internas.
11. No habilites ProxyRequests On salvo caso muy específico y seguro.
12. Configura logs separados por sitio.
13. Usa headers de seguridad.
14. Habilita compresión y cache para assets.
15. Usa mpm_event + PHP-FPM si usas PHP moderno.
16. Protege /server-status.
17. No uses permisos 777.
18. Monitorea errores 5xx, latencia, disco y certificados.
19. Versiona configuración no secreta.
20. Define rollback para cambios de producción.
```

---

# Fuentes de referencia recomendadas

```text
- Ubuntu Server documentation: Apache2 installation and configuration.
- Apache HTTP Server 2.4 documentation.
- Apache Virtual Host documentation.
- Apache mod_proxy documentation.
- Apache Reverse Proxy Guide.
- Apache SSL/TLS documentation.
- Certbot instructions for Apache on Ubuntu.
- Let's Encrypt documentation.
- Ubuntu UFW documentation.
```
