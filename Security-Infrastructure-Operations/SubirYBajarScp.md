# SubirYBajarScp
Recordatorio para subir y bajar archivos a través de SSH

## Subir archivos desde tu PC a un servidor remoto
### Enviar un archivo a un servidor EC2
```sh
scp -i "TuLlavePem.pem" /ruta/local/archivo.txt ubuntu@<EC2_PUBLIC_IP>:/ruta/remota/
```
Ejemplo:
```sh
scp -i "TuLlavePem.pem" archivo.txt ubuntu@3.95.84.123:/home/ubuntu/
```
Esto enviará archivo.txt al servidor en la carpeta /home/ubuntu/.

### Subir una carpeta completa
```sh
scp -i "TuLlavePem.pem" -r /ruta/local/carpeta ubuntu@<EC2_PUBLIC_IP>:/ruta/remota/
```
Ejemplo:
```sh
scp -i "TuLlavePem.pem" -r mi_carpeta ubuntu@3.95.84.123:/home/ubuntu/
```
Esto subirá mi_carpeta con todo su contenido.

## Descargar archivos desde un servidor remoto a tu PC
### Descargar un archivo
```sh
scp -i "TuLlavePem.pem" ubuntu@<EC2_PUBLIC_IP>:/ruta/remota/archivo.txt /ruta/local/
```
Ejemplo:
```sh
scp -i "TuLlavePem.pem" ubuntu@3.95.84.123:/home/ubuntu/archivo.txt .
```
Esto traerá archivo.txt al directorio actual en tu PC.

### Descargar una carpeta completa
```sh
scp -i "TuLlavePem.pem" -r ubuntu@<EC2_PUBLIC_IP>:/ruta/remota/carpeta /ruta/local/
```
Ejemplo:
```sh
scp -i "TuLlavePem.pem" -r ubuntu@3.95.84.123:/home/ubuntu/datos .
```
Esto descargará la carpeta datos al directorio actual de tu PC.

## Consejos Adicionales
  * Asegúrate de que el puerto 22 esté abierto en la configuración de AWS (grupo de seguridad de EC2).
  * Si la conexión es lenta, puedes usar -C para comprimir la transferencia:
```sh
scp -C -i "TuLlavePem.pem" archivo.txt ubuntu@<EC2_PUBLIC_IP>:/home/ubuntu/
```
  * Puedes usar -P <puerto> si SSH usa un puerto diferente al 22:
```sh
scp -P 2222 -i "TuLlavePem.pem" archivo.txt usuario@<IP>:~
```
¡Y eso es todo! Ahora puedes transferir archivos de forma segura entre tu computadora y un servidor remoto con scp

## Llave para comunicar por SSH
Para crear un par de llaves privada y pública utilizar
```sh
ssh-keygen -t ed25519 -C farios@uc.cl
```
Que pedirá una passphrase como clave. A esta altura ya ni lo utilizo. En el caso de sistemas que no utilicen el algoritmo 25519:
```sh
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
Estos ejemplos crearán una llave privada y una pública en el directorio oculto .ssh. La llave privada no se comparte con nadie, en cambio la pública se envía a los administradores de los repositorios. Para ello, se puede acceder con:
```sh
cat ~/.ssh/id_rsa.pub
```
En el caso de usar ```rsa```. La salida de esa llave es la que se envía a las Deploy Key en los repositorios acá para que se terceros tengan permiso de utilizar la cuenta sin las credenciales de ésta.
Settings -> (Scurity)Desploy Keys.

Por último se debe inicializar el agente con 
```sh
eval "$(ssh-agent -s)"
```
