# Guía de uso de conexiones SSH

## Objetivo

Esta guía explica cómo usar SSH de forma práctica y segura para conectarse a servidores, copiar archivos, administrar llaves, usar PuTTY, configurar conexiones persistentes, crear túneles, usar Jupyter por SSH desde el navegador y acceder a aplicaciones internas sin exponerlas públicamente.

Está organizada en tres niveles:

```text
1. Básico: conexión con OpenSSH, llaves, scp, sftp, permisos y primeros comandos.
2. Intermedio: ~/.ssh/config, ssh-agent, PuTTY, ProxyJump, túneles, rsync, Jupyter y servicios web internos.
3. Avanzado: conexiones persistentes con ControlMaster, hardening del servidor, bastion hosts, auditoría, gobernanza de llaves, troubleshooting y patrones seguros.
```

La guía asume Linux/macOS como cliente principal con OpenSSH. También incluye Windows con OpenSSH y PuTTY.

---

# Parte I: fundamentos de SSH

## 1. Qué es SSH

SSH significa Secure Shell. Es un protocolo para acceder de forma segura a una máquina remota.

Usos comunes:

```text
Entrar a un servidor remoto.
Ejecutar comandos.
Copiar archivos.
Crear túneles seguros.
Administrar servidores.
Conectarse a bases de datos privadas.
Usar Jupyter remoto desde navegador local.
Acceder a dashboards internos.
Usar Git sobre SSH.
Conectar VS Code a servidores remotos.
```

SSH cifra el tráfico entre cliente y servidor. Esto protege contra lectura de datos, manipulación de sesión y robo de credenciales en tránsito.

---

## 2. Componentes principales

```text
Cliente SSH:
Máquina desde donde te conectas.

Servidor SSH:
Máquina remota que acepta conexiones.

sshd:
Servicio del servidor que escucha conexiones SSH.

ssh:
Cliente de línea de comandos.

scp:
Copia archivos sobre SSH.

sftp:
Transferencia interactiva de archivos sobre SSH.

ssh-keygen:
Genera y administra llaves.

ssh-agent:
Mantiene llaves privadas cargadas en memoria.

ssh-add:
Agrega llaves al agente.

authorized_keys:
Archivo remoto que contiene llaves públicas autorizadas.
```

---

## 3. Instalación de OpenSSH

### 3.1 Linux Ubuntu/Debian

Cliente:

```bash
sudo apt update
sudo apt install openssh-client
```

Servidor:

```bash
sudo apt install openssh-server
```

Ver estado del servidor:

```bash
sudo systemctl status ssh
```

Iniciar:

```bash
sudo systemctl start ssh
```

Habilitar al arranque:

```bash
sudo systemctl enable ssh
```

---

### 3.2 macOS

macOS incluye cliente SSH.

Ver versión:

```bash
ssh -V
```

Para habilitar servidor SSH:

```text
System Settings -> General -> Sharing -> Remote Login
```

---

### 3.3 Windows

Windows 10/11 y Windows Server modernos incluyen OpenSSH Client como característica opcional.

Ver en PowerShell:

```powershell
ssh -V
```

También puedes instalar o habilitar OpenSSH desde:

```text
Settings -> Apps -> Optional Features
```

O usar PuTTY, que se explica más adelante.

---

## 4. Primer comando SSH

Formato:

```bash
ssh usuario@host
```

Ejemplo:

```bash
ssh ubuntu@203.0.113.10
```

Con puerto específico:

```bash
ssh -p 2222 ubuntu@203.0.113.10
```

Con llave específica:

```bash
ssh -i ~/.ssh/id_ed25519 ubuntu@203.0.113.10
```

Con comando remoto:

```bash
ssh ubuntu@203.0.113.10 "hostname && uptime"
```

---

## 5. Primer contacto y host keys

La primera vez que te conectas a un servidor, SSH muestra la huella del host.

Ejemplo:

```text
The authenticity of host '203.0.113.10' can't be established.
ED25519 key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Esto verifica la identidad del servidor.

Si aceptas, se guarda en:

```bash
~/.ssh/known_hosts
```

Regla:

```text
No aceptes ciegamente una host key en ambientes críticos.
Valida la huella por un canal confiable.
```

Si cambia inesperadamente, SSH advertirá posible ataque o reinstalación del servidor.

---

## 6. Estructura de `~/.ssh`

Directorio local:

```bash
~/.ssh/
```

Archivos comunes:

```text
id_ed25519           llave privada Ed25519.
id_ed25519.pub       llave pública Ed25519.
id_rsa               llave privada RSA.
id_rsa.pub           llave pública RSA.
config               configuración del cliente SSH.
known_hosts          host keys conocidas.
authorized_keys      llaves autorizadas, en el servidor.
```

Permisos recomendados:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/config
```

En servidor:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

# Parte II: llaves SSH

## 7. Autenticación por contraseña vs llave

Autenticación por contraseña:

```text
Simple.
Fácil de entender.
Mayor riesgo ante fuerza bruta, phishing o reutilización de contraseñas.
```

Autenticación por llave:

```text
Más segura.
No envía contraseña.
Permite desactivar login por password.
Puede protegerse con passphrase.
Escala mejor para administración.
```

Recomendación:

```text
Usa llaves SSH con passphrase.
Deshabilita contraseña en servidores productivos cuando tengas acceso por llaves verificado.
```

---

## 8. Llave pública y llave privada

Una llave SSH tiene dos partes:

```text
Llave privada:
Se queda en tu máquina.
No se comparte.
Debe protegerse.

Llave pública:
Se puede copiar al servidor.
Se guarda en authorized_keys.
```

Regla:

```text
Nunca compartas la llave privada.
Nunca la subas a Git.
Nunca la pegues en tickets, chats o emails.
```

---

## 9. Tipos de llaves

Tipos comunes:

```text
Ed25519:
Recomendado para la mayoría de usos modernos.
Corta, rápida y segura.

RSA:
Compatible con sistemas antiguos.
Usar 3072 o 4096 bits.

ECDSA:
Menos usado actualmente.

FIDO/U2F security key:
Llaves respaldadas por hardware, como YubiKey.
```

Recomendación general:

```text
Usa Ed25519 salvo que un sistema antiguo exija RSA.
```

---

## 10. Generar llave Ed25519

```bash
ssh-keygen -t ed25519 -a 100 -C "felipe@laptop-2026"
```

Guardar en ruta sugerida:

```text
~/.ssh/id_ed25519
```

Usa passphrase cuando lo pida.

Parámetros:

```text
-t ed25519:
Tipo de llave.

-a 100:
Número de rondas KDF para proteger la llave privada.

-C:
Comentario para identificar la llave.
```

Ver archivos:

```bash
ls -la ~/.ssh/id_ed25519*
```

---

## 11. Generar llave RSA

Solo si necesitas compatibilidad:

```bash
ssh-keygen -t rsa -b 4096 -o -a 100 -C "felipe@laptop-2026"
```

Parámetros:

```text
-b 4096:
Tamaño de llave RSA.

-o:
Usa formato privado OpenSSH moderno.

-a 100:
Endurece derivación de clave para passphrase.
```

---

## 12. Generar llave en formato PEM

A veces proveedores o sistemas antiguos piden “PEM”. En SSH, esto suele generar confusión porque se mezclan:

```text
Tipo de algoritmo:
RSA, Ed25519, ECDSA.

Formato de archivo:
OpenSSH, PEM, PKCS8, RFC4716, PPK.
```

Para generar una llave RSA privada en formato PEM:

```bash
ssh-keygen -t rsa -b 4096 -m PEM -f ~/.ssh/id_rsa_pem -C "felipe@pem"
```

Esto crea:

```text
~/.ssh/id_rsa_pem
~/.ssh/id_rsa_pem.pub
```

Permisos:

```bash
chmod 600 ~/.ssh/id_rsa_pem
```

Uso:

```bash
ssh -i ~/.ssh/id_rsa_pem usuario@host
```

Notas:

```text
PEM suele usarse con RSA.
Ed25519 normalmente usa formato OpenSSH.
AWS EC2 suele entregar archivos .pem, que son llaves privadas en formato consumible por OpenSSH.
```

---

## 13. Convertir llave privada OpenSSH a PEM

Cuidado: haz backup antes.

```bash
cp ~/.ssh/id_rsa ~/.ssh/id_rsa.backup
ssh-keygen -p -m PEM -f ~/.ssh/id_rsa
```

Para volver a formato OpenSSH moderno:

```bash
ssh-keygen -p -o -f ~/.ssh/id_rsa
```

Regla:

```text
Convierte formatos solo cuando un sistema externo lo exige.
Para uso normal con OpenSSH moderno, mantén formato OpenSSH.
```

---

## 14. Obtener llave pública desde llave privada

Si perdiste el `.pub`, puedes regenerarlo:

```bash
ssh-keygen -y -f ~/.ssh/id_ed25519 > ~/.ssh/id_ed25519.pub
```

Para PEM:

```bash
ssh-keygen -y -f ~/.ssh/id_rsa_pem > ~/.ssh/id_rsa_pem.pub
```

---

## 15. Ver fingerprint de una llave

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```

Con SHA256, normalmente por defecto:

```text
256 SHA256:... felipe@laptop-2026 (ED25519)
```

Ver randomart:

```bash
ssh-keygen -lvf ~/.ssh/id_ed25519.pub
```

Uso:

```text
Identificar llaves.
Auditar authorized_keys.
Confirmar que una llave pública corresponde a una privada.
```

---

## 16. Cambiar passphrase de una llave

```bash
ssh-keygen -p -f ~/.ssh/id_ed25519
```

Esto no cambia la llave pública ni requiere modificar `authorized_keys`.

---

# Parte III: instalar llaves en servidores

## 17. Usar `ssh-copy-id`

Si el servidor permite login por contraseña temporalmente:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub usuario@host
```

Con puerto:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 usuario@host
```

Luego prueba:

```bash
ssh -i ~/.ssh/id_ed25519 usuario@host
```

`ssh-copy-id` agrega tu llave pública al archivo remoto:

```bash
~/.ssh/authorized_keys
```

---

## 18. Copiar llave manualmente

En local:

```bash
cat ~/.ssh/id_ed25519.pub
```

En servidor:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Pega la llave pública completa en una sola línea.

Formato típico:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... felipe@laptop-2026
```

---

## 19. Probar autenticación por llave

```bash
ssh -i ~/.ssh/id_ed25519 usuario@host
```

Modo verboso:

```bash
ssh -vvv -i ~/.ssh/id_ed25519 usuario@host
```

El modo `-vvv` ayuda a diagnosticar:

```text
Qué llaves se ofrecen.
Qué método falla.
Qué archivo de configuración se usa.
Qué host key se valida.
```

---

## 20. Deshabilitar contraseña en servidor

Hazlo solo después de comprobar que puedes entrar con llave.

Archivo:

```bash
sudo nano /etc/ssh/sshd_config
```

Configuración típica:

```sshconfig
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitRootLogin no
```

Validar config:

```bash
sudo sshd -t
```

Recargar:

```bash
sudo systemctl reload ssh
```

Mantén una sesión abierta mientras pruebas otra nueva. Si algo falla, podrás revertir.

---

## 21. Usuarios permitidos

Restringir acceso:

```sshconfig
AllowUsers ubuntu deploy felipe
```

O por grupos:

```sshconfig
AllowGroups ssh-users
```

Crear grupo:

```bash
sudo groupadd ssh-users
sudo usermod -aG ssh-users felipe
```

Regla:

```text
No todos los usuarios del sistema necesitan acceso SSH.
```

---

# Parte IV: conexión con OpenSSH

## 22. Conexión básica

```bash
ssh usuario@servidor
```

Ejemplos:

```bash
ssh ubuntu@203.0.113.10
ssh deploy@app.example.com
ssh -p 2222 deploy@app.example.com
ssh -i ~/.ssh/prod_ed25519 deploy@app.example.com
```

Ejecutar comando remoto:

```bash
ssh deploy@app.example.com "sudo systemctl status myapp"
```

Entrar con pseudo-terminal forzada:

```bash
ssh -t deploy@app.example.com "sudo journalctl -u myapp -f"
```

---

## 23. `~/.ssh/config`

Evita escribir comandos largos.

Archivo:

```bash
nano ~/.ssh/config
chmod 600 ~/.ssh/config
```

Ejemplo:

```sshconfig
Host prod
    HostName 203.0.113.10
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/prod_ed25519
    IdentitiesOnly yes
```

Uso:

```bash
ssh prod
scp file.txt prod:/tmp/
```

Ventajas:

```text
Alias claros.
Menos errores.
Llaves por servidor.
Puertos configurados.
Túneles predefinidos.
ProxyJump.
Conexiones persistentes.
```

---

## 24. Configuración por patrones

```sshconfig
Host *.internal
    User ubuntu
    IdentityFile ~/.ssh/internal_ed25519
    IdentitiesOnly yes
```

```sshconfig
Host github.com
    User git
    IdentityFile ~/.ssh/github_ed25519
    IdentitiesOnly yes
```

Orden:

```text
OpenSSH aplica la primera coincidencia para cada parámetro.
Pon reglas específicas antes que reglas generales.
```

---

## 25. Opciones útiles del cliente

```sshconfig
Host *
    ServerAliveInterval 30
    ServerAliveCountMax 3
    AddKeysToAgent yes
    IdentityAgent SSH_AUTH_SOCK
```

Significado:

```text
ServerAliveInterval:
Envía señales para detectar conexiones muertas.

ServerAliveCountMax:
Corta si no hay respuesta tras varios intentos.

AddKeysToAgent:
Agrega llaves al agente cuando se usan.

IdentitiesOnly:
Usa solo la llave indicada y no todas las del agente.
```

---

# Parte V: ssh-agent

## 26. Qué es ssh-agent

`ssh-agent` guarda llaves privadas desbloqueadas en memoria para no escribir la passphrase en cada conexión.

Ver agente:

```bash
echo "$SSH_AUTH_SOCK"
```

Agregar llave:

```bash
ssh-add ~/.ssh/id_ed25519
```

Listar llaves cargadas:

```bash
ssh-add -l
```

Eliminar una llave:

```bash
ssh-add -d ~/.ssh/id_ed25519
```

Eliminar todas:

```bash
ssh-add -D
```

---

## 27. Tiempo de vida de una llave en agente

Agregar con expiración:

```bash
ssh-add -t 4h ~/.ssh/id_ed25519
```

Esto limita la exposición si la sesión queda abierta.

---

## 28. Agent forwarding

Permite usar tu agente local desde un servidor intermedio.

Comando:

```bash
ssh -A bastion
```

Config:

```sshconfig
Host bastion
    ForwardAgent yes
```

Advertencia:

```text
Agent forwarding puede ser riesgoso si el servidor remoto no es confiable.
Un atacante con control del servidor puede usar tu agente mientras la sesión esté activa.
```

Preferencia:

```text
Usa ProxyJump en vez de agent forwarding cuando solo necesitas saltar a otro host.
```

---

# Parte VI: transferencia de archivos

## 29. `scp`

`scp` copia archivos entre hosts usando SSH.

Subir archivo:

```bash
scp archivo.txt usuario@host:/ruta/remota/
```

Descargar archivo:

```bash
scp usuario@host:/ruta/remota/archivo.txt .
```

Usar llave:

```bash
scp -i ~/.ssh/id_ed25519 archivo.txt usuario@host:/tmp/
```

Usar puerto:

```bash
scp -P 2222 archivo.txt usuario@host:/tmp/
```

Nota:

```text
En scp, el puerto usa -P mayúscula.
En ssh, el puerto usa -p minúscula.
```

Copiar directorio:

```bash
scp -r ./carpeta usuario@host:/opt/
```

Preservar timestamps y permisos:

```bash
scp -p archivo.txt usuario@host:/tmp/
```

Compresión:

```bash
scp -C archivo_grande.csv usuario@host:/tmp/
```

---

## 30. `scp` con alias de config

Con `~/.ssh/config`:

```sshconfig
Host prod
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/prod_ed25519
```

Puedes usar:

```bash
scp archivo.txt prod:/tmp/
scp prod:/var/log/myapp/app.log .
```

---

## 31. Copiar entre servidores remotos

```bash
scp usuario1@host1:/tmp/file.txt usuario2@host2:/tmp/
```

En muchos casos conviene descargar y subir desde local, o usar `rsync` con control explícito.

---

## 32. `sftp`

Conexión:

```bash
sftp usuario@host
```

Comandos dentro de SFTP:

```text
pwd        ruta remota
lpwd       ruta local
ls         listar remoto
lls        listar local
cd         cambiar remoto
lcd        cambiar local
put file   subir
get file   descargar
mput *.csv subir varios
mget *.log descargar varios
mkdir dir  crear directorio remoto
rm file    borrar remoto
exit       salir
```

Subir archivo directo:

```bash
sftp usuario@host:/ruta/remota/ <<< $'put archivo.txt'
```

---

## 33. `rsync` sobre SSH

`rsync` es mejor para sincronizar carpetas.

Subir:

```bash
rsync -avz ./dist/ usuario@host:/var/www/app/
```

Con llave:

```bash
rsync -avz -e "ssh -i ~/.ssh/prod_ed25519" ./dist/ usuario@host:/var/www/app/
```

Con puerto:

```bash
rsync -avz -e "ssh -p 2222" ./dist/ usuario@host:/var/www/app/
```

Eliminar archivos remotos que ya no existen localmente:

```bash
rsync -avz --delete ./dist/ usuario@host:/var/www/app/
```

Simulación:

```bash
rsync -avz --dry-run ./dist/ usuario@host:/var/www/app/
```

Regla:

```text
Usa scp para copias simples.
Usa sftp para exploración interactiva.
Usa rsync para despliegues y sincronización incremental.
```

---

# Parte VII: PuTTY en Windows

## 34. Qué es PuTTY

PuTTY es un cliente SSH gráfico para Windows. Incluye herramientas como:

```text
PuTTY:
Cliente SSH.

PuTTYgen:
Generador y conversor de llaves.

Pageant:
Agente de llaves.

PSCP:
Copia de archivos tipo scp.

PSFTP:
Cliente SFTP.
```

Windows moderno también puede usar OpenSSH nativo, pero PuTTY sigue siendo común en entornos corporativos.

---

## 35. Crear llave con PuTTYgen

Pasos:

```text
1. Abrir PuTTYgen.
2. Elegir tipo de llave.
3. Generar la llave moviendo el mouse si lo solicita.
4. Agregar Key comment.
5. Agregar passphrase.
6. Guardar private key como .ppk.
7. Copiar public key en formato authorized_keys.
```

Recomendación:

```text
Usa Ed25519 si tu versión de PuTTY y servidor lo soportan.
Usa RSA 4096 si necesitas compatibilidad amplia.
```

---

## 36. Instalar llave pública de PuTTY en servidor

PuTTYgen muestra una sección como:

```text
Public key for pasting into OpenSSH authorized_keys file
```

Copia esa línea completa.

En servidor:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Pega la llave en una sola línea.

---

## 37. Conectar con PuTTY usando llave

En PuTTY:

```text
Session:
- Host Name: usuario@host
- Port: 22
- Connection type: SSH

Connection -> SSH -> Auth -> Credentials:
- Private key file for authentication: archivo .ppk

Session:
- Saved Sessions: nombre
- Save
```

Luego:

```text
Open
```

---

## 38. Convertir PEM/OpenSSH a PPK

Abrir PuTTYgen:

```text
File -> Load private key
```

Selecciona:

```text
.pem
id_rsa
id_ed25519
```

Luego:

```text
Save private key
```

Guarda como:

```text
key.ppk
```

---

## 39. Convertir PPK a OpenSSH

En PuTTYgen:

```text
Conversions -> Export OpenSSH key
```

Guarda como:

```text
id_ed25519
```

Luego en Linux/macOS/WSL:

```bash
chmod 600 ~/.ssh/id_ed25519
```

---

## 40. PSCP y PSFTP

PSCP:

```powershell
pscp -i C:\keys\prod.ppk file.txt user@host:/tmp/
```

Descargar:

```powershell
pscp -i C:\keys\prod.ppk user@host:/tmp/file.txt .
```

PSFTP:

```powershell
psftp -i C:\keys\prod.ppk user@host
```

---

# Parte VIII: túneles SSH

## 41. Qué es un túnel SSH

Un túnel SSH reenvía tráfico de un puerto a través de una conexión SSH cifrada.

Usos:

```text
Acceder a Jupyter remoto desde navegador local.
Entrar a una base de datos privada.
Acceder a un dashboard interno.
Probar una API interna.
Usar un servidor como proxy SOCKS.
Exponer temporalmente un servicio local en un servidor remoto.
```

Tipos:

```text
Local forwarding:
-L

Remote forwarding:
-R

Dynamic forwarding:
-D
```

---

## 42. Local port forwarding `-L`

Formato:

```bash
ssh -L puerto_local:host_destino:puerto_destino usuario@servidor_ssh
```

Ejemplo: acceder a PostgreSQL remoto que solo escucha en localhost del servidor:

```bash
ssh -L 5433:127.0.0.1:5432 ubuntu@db-server
```

Luego desde local:

```bash
psql -h 127.0.0.1 -p 5433 -U app_user -d app_db
```

Interpretación:

```text
localhost:5433 en tu máquina
se reenvía a 127.0.0.1:5432 visto desde db-server.
```

---

## 43. Mantener solo túnel sin shell

Usa `-N`:

```bash
ssh -N -L 5433:127.0.0.1:5432 ubuntu@db-server
```

Usa `-f` para enviar a background después de autenticar:

```bash
ssh -fN -L 5433:127.0.0.1:5432 ubuntu@db-server
```

Cuidado:

```text
Los túneles en background pueden quedar vivos.
Usa ps, pkill o ControlMaster para gestionarlos.
```

---

## 44. Remote port forwarding `-R`

Expone un puerto local en el servidor remoto.

Formato:

```bash
ssh -R puerto_remoto:host_local:puerto_local usuario@servidor
```

Ejemplo:

```bash
ssh -R 9000:127.0.0.1:3000 ubuntu@public-server
```

Esto permite que desde `public-server` se acceda a tu servicio local:

```bash
curl http://127.0.0.1:9000
```

Uso:

```text
Demos temporales.
Depuración.
Conectar servicios internos a un servidor intermedio.
```

Advertencia:

```text
Remote forwarding puede exponer servicios locales.
Revisa GatewayPorts y permisos del servidor.
```

---

## 45. Dynamic forwarding `-D`

Crea un proxy SOCKS local.

```bash
ssh -N -D 1080 ubuntu@server
```

Configura navegador o herramienta para usar:

```text
SOCKS5 proxy: 127.0.0.1:1080
```

Uso:

```text
Navegar como si estuvieras desde la red del servidor.
Acceder a paneles internos.
Depurar restricciones por IP.
```

Advertencia:

```text
No uses túneles dinámicos para evadir políticas de red.
Debe haber autorización.
```

---

## 46. Túneles en `~/.ssh/config`

Puedes preconfigurar túneles.

```sshconfig
Host db-tunnel
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/prod_ed25519
    LocalForward 5433 127.0.0.1:5432
    ExitOnForwardFailure yes
```

Uso:

```bash
ssh -N db-tunnel
```

Para Jupyter:

```sshconfig
Host jupyter-prod
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/prod_ed25519
    LocalForward 8888 127.0.0.1:8888
    ExitOnForwardFailure yes
```

Uso:

```bash
ssh -N jupyter-prod
```

---

# Parte IX: Jupyter seguro por SSH

## 47. Problema

Jupyter corre como aplicación web. Exponerlo directamente a internet puede ser riesgoso porque permite ejecutar código en el servidor.

Patrón seguro para uso personal o de desarrollo:

```text
Jupyter escucha solo en 127.0.0.1 del servidor.
SSH crea túnel local.
Navegador local abre http://127.0.0.1:8888.
```

---

## 48. Instalar Jupyter en servidor

Crear entorno:

```bash
python3 -m venv ~/venvs/jupyter
source ~/venvs/jupyter/bin/activate
pip install jupyterlab
```

Ejecutar:

```bash
jupyter lab --no-browser --ip=127.0.0.1 --port=8888
```

Salida mostrará una URL con token:

```text
http://127.0.0.1:8888/lab?token=...
```

---

## 49. Crear túnel desde local

En tu máquina local:

```bash
ssh -N -L 8888:127.0.0.1:8888 usuario@servidor
```

Luego abre en tu navegador local:

```text
http://127.0.0.1:8888/lab
```

Pega el token que aparece en el servidor.

---

## 50. Evitar exponer Jupyter

Evita:

```bash
jupyter lab --ip=0.0.0.0 --port=8888
```

A menos que configures correctamente:

```text
HTTPS.
Password/token fuerte.
Reverse proxy.
Firewall.
Autenticación.
Usuarios.
Restricciones de red.
```

Recomendación:

```text
Para uso individual, usa túnel SSH.
Para múltiples usuarios, usa JupyterHub o una plataforma administrada.
```

---

## 51. Jupyter con `~/.ssh/config`

Config:

```sshconfig
Host gpu-notebook
    HostName 203.0.113.20
    User ubuntu
    IdentityFile ~/.ssh/gpu_ed25519
    LocalForward 8888 127.0.0.1:8888
    ServerAliveInterval 30
    ServerAliveCountMax 3
    ExitOnForwardFailure yes
```

En servidor:

```bash
jupyter lab --no-browser --ip=127.0.0.1 --port=8888
```

En local:

```bash
ssh -N gpu-notebook
```

Navegador:

```text
http://127.0.0.1:8888/lab
```

---

## 52. Jupyter con systemd de usuario

En servidor, puedes crear un servicio de usuario.

Archivo:

```bash
mkdir -p ~/.config/systemd/user
nano ~/.config/systemd/user/jupyter.service
```

Contenido:

```ini
[Unit]
Description=Jupyter Lab user service

[Service]
Type=simple
WorkingDirectory=%h
ExecStart=%h/venvs/jupyter/bin/jupyter lab --no-browser --ip=127.0.0.1 --port=8888
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

Activar:

```bash
systemctl --user daemon-reload
systemctl --user enable --now jupyter
```

Ver logs:

```bash
journalctl --user -u jupyter -f
```

Para permitir que siga activo sin sesión interactiva:

```bash
loginctl enable-linger "$USER"
```

Cuidado:

```text
Jupyter tendrá acceso a archivos del usuario.
Protege el usuario, la llave SSH y el token.
```

---

# Parte X: aplicaciones similares por SSH

## 53. Acceso a dashboards internos

Ejemplo: Grafana remoto en `127.0.0.1:3000`.

```bash
ssh -N -L 3000:127.0.0.1:3000 ubuntu@server
```

Navegador local:

```text
http://127.0.0.1:3000
```

---

## 54. Acceso a API interna

API en servidor:

```text
127.0.0.1:8000
```

Túnel:

```bash
ssh -N -L 8000:127.0.0.1:8000 ubuntu@server
```

Local:

```bash
curl http://127.0.0.1:8000/health
```

---

## 55. Acceso a PostgreSQL privado

Túnel:

```bash
ssh -N -L 5433:127.0.0.1:5432 ubuntu@server
```

Conectar:

```bash
psql -h 127.0.0.1 -p 5433 -U app_user -d app_db
```

---

## 56. Acceso a MySQL privado

```bash
ssh -N -L 3307:127.0.0.1:3306 ubuntu@server
```

```bash
mysql -h 127.0.0.1 -P 3307 -u app_user -p
```

---

## 57. Acceso a Redis privado

```bash
ssh -N -L 6380:127.0.0.1:6379 ubuntu@server
```

```bash
redis-cli -h 127.0.0.1 -p 6380
```

---

## 58. VS Code Remote SSH

VS Code Remote SSH permite desarrollar en un servidor remoto usando SSH.

Requisitos:

```text
VS Code local.
Extensión Remote - SSH.
Acceso SSH funcional.
Permisos adecuados en servidor.
```

Config:

```sshconfig
Host dev-gpu
    HostName 203.0.113.20
    User ubuntu
    IdentityFile ~/.ssh/gpu_ed25519
```

En VS Code:

```text
Command Palette -> Remote-SSH: Connect to Host -> dev-gpu
```

Buenas prácticas:

```text
No desarrolles como root.
Usa entornos virtuales.
No guardes secretos en workspace.
Controla extensiones instaladas remotamente.
Cierra sesiones inactivas.
```

---

## 59. Git por SSH

Generar llave dedicada:

```bash
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/github_ed25519 -C "felipe@github"
```

Config:

```sshconfig
Host github.com
    User git
    HostName github.com
    IdentityFile ~/.ssh/github_ed25519
    IdentitiesOnly yes
```

Probar:

```bash
ssh -T git@github.com
```

Clonar:

```bash
git clone git@github.com:org/repo.git
```

Regla:

```text
Usa llaves separadas para Git, servidores personales y producción.
```

---

# Parte XI: bastion hosts y ProxyJump

## 60. Qué es un bastion

Un bastion host es un servidor de salto. Permite entrar a una red privada sin exponer todos los servidores.

Patrón:

```text
Local -> Bastion público -> Servidor privado
```

Ejemplo:

```text
Laptop -> bastion.example.com -> 10.0.1.25
```

---

## 61. Conectar con ProxyJump

Comando:

```bash
ssh -J ubuntu@bastion.example.com ubuntu@10.0.1.25
```

Con llave:

```bash
ssh -i ~/.ssh/prod_ed25519 -J ubuntu@bastion.example.com ubuntu@10.0.1.25
```

---

## 62. ProxyJump en config

```sshconfig
Host bastion
    HostName bastion.example.com
    User ubuntu
    IdentityFile ~/.ssh/bastion_ed25519

Host app-private
    HostName 10.0.1.25
    User ubuntu
    IdentityFile ~/.ssh/app_ed25519
    ProxyJump bastion
```

Uso:

```bash
ssh app-private
scp file.txt app-private:/tmp/
```

Ventaja:

```text
No necesitas copiar llaves privadas al bastion.
```

---

## 63. Túnel a través de bastion

Ejemplo: PostgreSQL en servidor privado.

```sshconfig
Host db-private
    HostName 10.0.2.15
    User ubuntu
    IdentityFile ~/.ssh/db_ed25519
    ProxyJump bastion
    LocalForward 5433 127.0.0.1:5432
    ExitOnForwardFailure yes
```

Uso:

```bash
ssh -N db-private
```

Local:

```bash
psql -h 127.0.0.1 -p 5433 -U app_user -d app_db
```

---

# Parte XII: conexiones persistentes y preconfiguradas

## 64. Qué son conexiones preconfiguradas

En la práctica, “conexiones pregeneradas” suele referirse a:

```text
Aliases en ~/.ssh/config.
Sesiones guardadas en PuTTY.
Túneles definidos en la configuración.
Multiplexing con ControlMaster.
Scripts de conexión.
systemd user services para túneles.
```

No se “genera” una conexión antes de usarla; se preconfiguran parámetros para abrirla de forma repetible.

---

## 65. ControlMaster

OpenSSH puede reutilizar una conexión TCP existente para abrir sesiones nuevas más rápido.

Config:

```sshconfig
Host *
    ControlMaster auto
    ControlPath ~/.ssh/control-%r@%h:%p
    ControlPersist 10m
```

Significado:

```text
ControlMaster auto:
Crea o reutiliza una conexión maestra.

ControlPath:
Socket local de control.

ControlPersist 10m:
Mantiene la conexión maestra abierta 10 minutos después de cerrar la última sesión.
```

Uso:

```bash
ssh prod
ssh prod "uptime"
scp file.txt prod:/tmp/
```

Las conexiones posteriores reutilizan la conexión maestra.

---

## 66. Directorio para ControlPath

Evita problemas de permisos:

```bash
mkdir -p ~/.ssh/control
chmod 700 ~/.ssh/control
```

Config:

```sshconfig
Host *
    ControlMaster auto
    ControlPath ~/.ssh/control/%C
    ControlPersist 10m
```

`%C` genera un hash de conexión, más robusto que `%r@%h:%p`.

---

## 67. Gestionar conexión maestra

Verificar:

```bash
ssh -O check prod
```

Cerrar conexión maestra:

```bash
ssh -O exit prod
```

Forzar cierre:

```bash
ssh -O stop prod
```

---

## 68. Túneles persistentes con systemd de usuario

En tu máquina local, puedes crear un servicio de usuario para mantener un túnel.

Archivo:

```bash
mkdir -p ~/.config/systemd/user
nano ~/.config/systemd/user/jupyter-tunnel.service
```

Contenido:

```ini
[Unit]
Description=SSH tunnel for remote Jupyter
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/ssh -N jupyter-prod
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Activar:

```bash
systemctl --user daemon-reload
systemctl --user enable --now jupyter-tunnel
```

Logs:

```bash
journalctl --user -u jupyter-tunnel -f
```

Requiere que `jupyter-prod` esté definido en `~/.ssh/config`.

---

## 69. autossh

`autossh` reinicia túneles si caen.

Instalar:

```bash
sudo apt install autossh
```

Uso:

```bash
autossh -M 0 -N -L 8888:127.0.0.1:8888 jupyter-prod
```

`-M 0` desactiva puerto de monitoreo propio; se puede combinar con `ServerAliveInterval`.

Config:

```sshconfig
Host jupyter-prod
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/prod_ed25519
    LocalForward 8888 127.0.0.1:8888
    ServerAliveInterval 30
    ServerAliveCountMax 3
    ExitOnForwardFailure yes
```

---

# Parte XIII: seguridad del servidor SSH

## 70. Configuración básica de `sshd_config`

Archivo:

```bash
sudo nano /etc/ssh/sshd_config
```

Configuración recomendada general:

```sshconfig
Port 22
Protocol 2
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
X11Forwarding no
AllowTcpForwarding yes
ClientAliveInterval 300
ClientAliveCountMax 2
MaxAuthTries 3
AllowUsers ubuntu deploy
```

Validar:

```bash
sudo sshd -t
```

Recargar:

```bash
sudo systemctl reload ssh
```

Advertencia:

```text
No cierres tu sesión actual hasta probar una nueva conexión.
```

---

## 71. Cambiar puerto SSH

Cambiar puerto no reemplaza seguridad, pero puede reducir ruido de bots.

```sshconfig
Port 2222
```

Firewall:

```bash
sudo ufw allow 2222/tcp
```

Validar y recargar:

```bash
sudo sshd -t
sudo systemctl reload ssh
```

Probar nueva conexión:

```bash
ssh -p 2222 usuario@host
```

Después de confirmar, si corresponde:

```bash
sudo ufw delete allow 22/tcp
```

---

## 72. Firewall

UFW:

```bash
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status verbose
```

Si usas puerto custom:

```bash
sudo ufw allow 2222/tcp
```

Restringir por IP:

```bash
sudo ufw allow from 203.0.113.50 to any port 22 proto tcp
```

Regla:

```text
En producción, limita SSH a redes confiables, VPN o bastion cuando sea posible.
```

---

## 73. Fail2ban

Instalar:

```bash
sudo apt install fail2ban
```

Config básica:

```bash
sudo nano /etc/fail2ban/jail.local
```

Contenido:

```ini
[sshd]
enabled = true
port = ssh
logpath = %(sshd_log)s
maxretry = 5
bantime = 1h
```

Reiniciar:

```bash
sudo systemctl restart fail2ban
```

Estado:

```bash
sudo fail2ban-client status sshd
```

---

## 74. Restringir forwarding

Si no quieres permitir túneles:

```sshconfig
AllowTcpForwarding no
PermitTunnel no
X11Forwarding no
```

Si solo algunos usuarios pueden tunelar, usa bloques `Match`.

Ejemplo:

```sshconfig
AllowTcpForwarding no

Match User datauser
    AllowTcpForwarding local
```

Regla:

```text
Los túneles son potentes.
Permítelos solo donde sean necesarios.
```

---

## 75. Comandos restringidos en authorized_keys

Puedes limitar una llave a un comando específico.

En `authorized_keys`:

```text
command="/usr/local/bin/backup-only",no-agent-forwarding,no-X11-forwarding,no-pty ssh-ed25519 AAAA...
```

Opciones útiles:

```text
from="203.0.113.10"
command="..."
no-agent-forwarding
no-port-forwarding
no-X11-forwarding
no-pty
```

Uso:

```text
Backups automatizados.
Deploy limitado.
Integraciones.
SFTP restringido.
```

---

# Parte XIV: gobernanza de llaves

## 76. Inventario de llaves

Toda organización debería saber:

```text
Quién posee cada llave.
Dónde está autorizada.
Para qué sirve.
Cuándo fue creada.
Cuándo expira o debe rotarse.
Qué permisos concede.
```

Campos mínimos:

```text
fingerprint.
owner.
email.
created_at.
last_review.
systems.
purpose.
status.
```

---

## 77. Separar llaves por propósito

No uses una sola llave para todo.

Ejemplo:

```text
~/.ssh/github_ed25519
~/.ssh/prod_bastion_ed25519
~/.ssh/staging_ed25519
~/.ssh/personal_vps_ed25519
```

Ventajas:

```text
Revocación más simple.
Menor impacto si se filtra una llave.
Auditoría más clara.
```

---

## 78. Rotación de llaves

Proceso:

```text
1. Generar nueva llave.
2. Instalar llave pública nueva.
3. Probar acceso.
4. Retirar llave antigua de authorized_keys.
5. Actualizar inventario.
6. Destruir copia privada antigua si corresponde.
```

Nunca elimines una llave antigua antes de confirmar que la nueva funciona.

---

## 79. Revocar acceso

Para revocar:

```bash
nano ~/.ssh/authorized_keys
```

Elimina la línea correspondiente.

Para identificar:

```bash
ssh-keygen -lf ~/.ssh/authorized_keys
```

Si no sabes cuál es:

```text
Compara fingerprint con inventario.
Usa comentarios claros en las llaves públicas.
```

---

# Parte XV: troubleshooting

## 80. Permiso denegado: publickey

Error:

```text
Permission denied (publickey).
```

Diagnóstico:

```bash
ssh -vvv usuario@host
```

Revisar:

```text
Usuario correcto.
Host correcto.
Llave correcta.
Permisos de ~/.ssh.
Llave pública en authorized_keys.
sshd_config permite PubkeyAuthentication.
No hay demasiadas llaves ofrecidas.
```

Soluciones comunes:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 600 ~/.ssh/config
```

En servidor:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Usar llave explícita:

```bash
ssh -i ~/.ssh/id_ed25519 -o IdentitiesOnly=yes usuario@host
```

---

## 81. Host key changed

Error:

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

Causas:

```text
Servidor reinstalado.
IP reutilizada.
Host key rotada.
Posible ataque man-in-the-middle.
```

No borres la entrada sin investigar.

Si confirmas que el cambio es legítimo:

```bash
ssh-keygen -R host
ssh-keygen -R 203.0.113.10
```

Luego reconecta y valida nueva huella.

---

## 82. Too many authentication failures

Causa:

```text
Tu agente ofrece demasiadas llaves antes de llegar a la correcta.
```

Solución:

```bash
ssh -i ~/.ssh/prod_ed25519 -o IdentitiesOnly=yes usuario@host
```

Config:

```sshconfig
Host prod
    HostName 203.0.113.10
    User ubuntu
    IdentityFile ~/.ssh/prod_ed25519
    IdentitiesOnly yes
```

---

## 83. Connection timed out

Causas:

```text
Servidor apagado.
Firewall bloquea.
Puerto incorrecto.
IP incorrecta.
SSH no escucha.
Ruta de red caída.
Security group cloud bloquea.
```

Diagnóstico:

```bash
ping host
nc -vz host 22
ssh -vvv usuario@host
```

En servidor, si tienes consola:

```bash
sudo systemctl status ssh
sudo ss -ltnp | grep ssh
sudo ufw status verbose
```

---

## 84. Connection refused

Causas:

```text
Host alcanzable, pero nada escucha en ese puerto.
sshd detenido.
Puerto cambiado.
Firewall rechazando activamente.
```

Diagnóstico servidor:

```bash
sudo systemctl status ssh
sudo ss -ltnp | grep ':22'
sudo journalctl -u ssh -n 100
```

---

## 85. Túnel no funciona

Revisar:

```text
Puerto local ocupado.
Destino remoto incorrecto.
Servicio destino escucha solo en otra interfaz.
Firewall.
ExitOnForwardFailure no configurado.
```

Comando útil:

```bash
ssh -vvv -N -L 8888:127.0.0.1:8888 usuario@host
```

Ver puerto local:

```bash
ss -ltnp | grep 8888
```

Probar desde servidor:

```bash
curl http://127.0.0.1:8888
```

---

## 86. Jupyter no abre

Checklist:

```text
[ ] Jupyter corre en servidor.
[ ] Jupyter escucha en 127.0.0.1:8888.
[ ] Túnel SSH está activo.
[ ] Puerto local no está ocupado.
[ ] Navegador usa http://127.0.0.1:8888.
[ ] Token correcto.
```

Servidor:

```bash
jupyter lab --no-browser --ip=127.0.0.1 --port=8888
```

Local:

```bash
ssh -N -L 8888:127.0.0.1:8888 usuario@host
```

---

# Parte XVI: checklists

## 87. Checklist de conexión básica

```text
[ ] Cliente OpenSSH instalado.
[ ] Host/IP correcto.
[ ] Usuario correcto.
[ ] Puerto correcto.
[ ] Llave privada existe.
[ ] Permisos de llave son 600.
[ ] Llave pública está en authorized_keys.
[ ] Firewall permite SSH.
[ ] ssh -vvv revisado si falla.
```

---

## 88. Checklist de servidor seguro

```text
[ ] PermitRootLogin no.
[ ] PubkeyAuthentication yes.
[ ] PasswordAuthentication no, si ya hay llaves probadas.
[ ] KbdInteractiveAuthentication no, si no se usa.
[ ] AllowUsers o AllowGroups configurado.
[ ] MaxAuthTries bajo.
[ ] UFW/security group restringido.
[ ] Fail2ban instalado si hay exposición pública.
[ ] Logs revisados.
[ ] Acceso de emergencia definido.
```

---

## 89. Checklist de llaves

```text
[ ] Llaves con passphrase.
[ ] Ed25519 por defecto.
[ ] RSA 4096 solo si hace falta compatibilidad.
[ ] Llaves separadas por propósito.
[ ] Comentarios claros.
[ ] Inventario de fingerprints.
[ ] Rotación definida.
[ ] Llaves antiguas revocadas.
[ ] Privadas nunca compartidas.
```

---

## 90. Checklist de Jupyter por SSH

```text
[ ] Jupyter escucha en 127.0.0.1.
[ ] No se expone 0.0.0.0 sin protección.
[ ] Túnel local creado con -L.
[ ] Token/password activo.
[ ] Usuario del servidor protegido.
[ ] Firewall no expone puerto 8888.
[ ] No se ejecuta como root.
[ ] Entorno virtual definido.
```

---

# Parte XVII: comandos rápidos

## 91. OpenSSH

```bash
ssh usuario@host
ssh -p 2222 usuario@host
ssh -i ~/.ssh/id_ed25519 usuario@host
ssh -vvv usuario@host
ssh usuario@host "uptime"
```

---

## 92. Llaves

```bash
ssh-keygen -t ed25519 -a 100 -C "user@machine"
ssh-keygen -t rsa -b 4096 -m PEM -f ~/.ssh/id_rsa_pem
ssh-keygen -y -f ~/.ssh/id_ed25519 > ~/.ssh/id_ed25519.pub
ssh-keygen -lf ~/.ssh/id_ed25519.pub
ssh-keygen -p -f ~/.ssh/id_ed25519
ssh-copy-id -i ~/.ssh/id_ed25519.pub usuario@host
```

---

## 93. Agent

```bash
ssh-add ~/.ssh/id_ed25519
ssh-add -l
ssh-add -D
ssh-add -t 4h ~/.ssh/id_ed25519
```

---

## 94. Archivos

```bash
scp archivo.txt usuario@host:/tmp/
scp usuario@host:/tmp/archivo.txt .
scp -r carpeta usuario@host:/tmp/
sftp usuario@host
rsync -avz ./dist/ usuario@host:/var/www/app/
```

---

## 95. Túneles

```bash
ssh -N -L 8888:127.0.0.1:8888 usuario@host
ssh -N -L 5433:127.0.0.1:5432 usuario@host
ssh -N -D 1080 usuario@host
ssh -R 9000:127.0.0.1:3000 usuario@host
```

---

## 96. ProxyJump

```bash
ssh -J usuario@bastion usuario@host-privado
scp -o ProxyJump=usuario@bastion file.txt usuario@host-privado:/tmp/
```

Config:

```sshconfig
Host private
    HostName 10.0.1.25
    User ubuntu
    ProxyJump bastion
```

---

## 97. ControlMaster

```sshconfig
Host *
    ControlMaster auto
    ControlPath ~/.ssh/control/%C
    ControlPersist 10m
```

Comandos:

```bash
ssh -O check prod
ssh -O exit prod
```

---

# Parte XVIII: resumen de reglas principales

```text
1. Usa llaves SSH, no contraseñas, para servidores productivos.
2. Usa Ed25519 salvo que necesites compatibilidad RSA.
3. Protege la llave privada con passphrase.
4. Usa chmod 600 en llaves privadas.
5. Copia solo la llave pública al servidor.
6. Usa ~/.ssh/config para evitar comandos largos y errores.
7. Usa IdentitiesOnly yes cuando tengas muchas llaves.
8. Usa ProxyJump en vez de copiar llaves a bastions.
9. Usa scp para copias simples, sftp para modo interactivo y rsync para sincronización.
10. Usa túneles -L para servicios privados como Jupyter, PostgreSQL o dashboards.
11. Ejecuta Jupyter en 127.0.0.1 y accede con túnel SSH.
12. No expongas Jupyter directamente a internet sin HTTPS, autenticación y configuración madura.
13. Usa ControlMaster para conexiones persistentes.
14. Usa systemd o autossh si necesitas túneles persistentes.
15. Deshabilita PermitRootLogin.
16. Deshabilita PasswordAuthentication solo después de probar llaves.
17. Restringe usuarios con AllowUsers o AllowGroups.
18. Audita authorized_keys regularmente.
19. Separa llaves por propósito.
20. No uses agent forwarding salvo necesidad y servidores confiables.
```

---

# Fuentes de referencia recomendadas

```text
- OpenSSH official website.
- OpenSSH manual: ssh.
- OpenSSH manual: ssh_config.
- OpenSSH manual: ssh-keygen.
- OpenSSH manual: scp.
- OpenSSH manual: sftp.
- OpenSSH manual: ssh-agent and ssh-add.
- OpenSSH manual: sshd_config.
- Jupyter Server documentation: Running a public Jupyter Server.
- Jupyter Notebook documentation.
- PuTTY and PuTTYgen documentation.
- Microsoft documentation: OpenSSH for Windows.
```
