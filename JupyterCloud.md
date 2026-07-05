# JupyterCloud
Conexiones externas para utilizar Jupyter en máquinas virtuales cloud con el browser en local
## Básico de instalación de python en máquinas Linux (ubuntu)
```sh 
sudo apt update && sudo apt install python3
sudo apt install python3-pip
python3 -m pip install --upgrade pip
```
## Básico de conexión de jupyter en ambiente virtual si es primera vez en un servidor cloud 
```sh 
python3 -m venv my_env
source my_env/bin/activate
```
O si la máquina es Windows
```sh 
my_env/Script/activate
```
Y se instala Juprterlab 
```sh 
pip install jupyterlab
```
## Disponibilizar el jupyter notebook sin uso de browser (máquina cloud, ejemplo con EC2 de AWS)
### Iniciar Jupyter Lab en la Instancia EC2
Se debe lanzar jupyter con la opción
```sh 
jupyter lab --ip=0.0.0.0 --port=8888 --no-browser
```
Donde:
  * ```--ip=0.0.0.0```: permite conexiones entrantes desde cualquier IP (no seguro, pero útil para probar cosas rápido)
  * ```--port=8888```: Usa el puerto 8888 (puedes cambiarlo).
  * ```--no-browser```: Evita que se abra un navegador en la instancia.

Jupyter generará una URL con un token como esta:
```sh 
http://127.0.0.1:8888/lab?token=xxxxxxxxxxxxxxxxxxxx
```
Pero este 127.0.0.1 no servirá desde tu computadora aún.

En el caso de que quieras una conexión rápida e insegura (no recomedado) puedes hacer:
```sh 
http://<IP_PUBLIA>:8888
``
Donde no es necesario agregar el token, pero para eso tienes que lanzarlo con esa opción. Y si lo quieres más inseguro aún, porque eres burro y trabajaste todo con el usuario root en tu máquina puedes hacer también:
```sh 
jupyter lab --ip=0.0.0.0 --port=8888 --no-browser --NotebookApp.token='' --allow-root
```

### Abrir el Puerto en la Seguridad de AWS
Ve a la consola de AWS → EC2.

Busca tu instancia y haz clic en Seguridad.

En Grupos de seguridad, haz clic en el grupo asociado a la instancia.

Edita las reglas de entrada y agrega una regla:

  * Tipo: Custom TCP
  * Puerto: 8888 (o el que elegiste)
  * Origen: ```My IP``` (para restringir el acceso solo a tu IP) o ```0.0.0.0/0``` (poco seguro, pero funciona para pruebas).

### Conectar desde tu Computador
Ahora, en tu navegador, accede a:
```sh 
http://<EC2_PUBLIC_IP>:8888/lab?token=xxxxxxxxxxxxxxxxxxxx
```
Reemplazando ```<EC2_PUBLIC_IP>``` con la IP pública de tu instancia (puedes verla en la consola de AWS en la sección de detalles de la instancia).
Si todo está bien, verás Jupyter Lab funcionando en tu navegador.
### (Opcional) Hacerlo Más Seguro con SSH Tunneling
Si no quieres abrir el puerto en AWS, puedes usar un túnel SSH para acceder:
Desde tu computador, ejecuta:
```sh 
ssh -L 8888:localhost:8888 -i "tu-clave.pem" ubuntu@<EC2_PUBLIC_IP>
```
Luego, accede en tu navegador a:
```sh 
http://localhost:8888/lab
```
Esto redirige el puerto de EC2 a tu PC, sin exponerlo en Internet.

En el caso de que tu llave ```PEM``` esté recién creada (sobretodo si la descargaste desde internet). Probablemente te aparezca el error: ```WARNING: UNPROTECTED PRIVATE KEY FILE!```

Para solucionar esto primero debes ir donde está localizada tu llave PEM u obtener su ruta.

Si estás en Windows abre PowerShell como administrador, y ejecuta:

```sh 
icacls "C:\Users\TuUsuario\TuLlavePem.pem" /inheritance:r
icacls "C:\Users\TuUsuario\TuLlavePem.pem" /grant:r %username%:R
```
Donde ```%username%``` lo reemplazas por tu nombre de usuario. Luego haz una prueba de conexión
```sh 
ssh -i "C:\Users\TuUsuario\TuLlavePem.pem" ubuntu@<EC2_PUBLIC_IP>
```

Si estás en Linux abre una terminal y navega hasta la carpeta donde está el archivo .pem:
```sh 
cd /ruta/del/archivo
```
Cambia los permisos del archivo para que solo el usuario actual tenga acceso:
```sh 
chmod 400 TuLlavePem.pem
```
Esto significa:

4 → Permiso de lectura para el dueño del archivo.
00 → Ningún permiso para otros usuarios.

Verifica los permisos del archivo con:
```sh 
ls -l TuLlavePem.pem
```
Debería mostrar algo como:
```sh 
-r-------- 1 tu_usuario tu_grupo 1692 feb 8 10:00 TuLlavePem.pem
```
Intenta conectarte de nuevo por SSH:
```sh
ssh -i TuLlavePem.pem ubuntu@<EC2_PUBLIC_IP>
```


