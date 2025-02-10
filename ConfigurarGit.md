# ConfigurarGit
Códigos para recordar como configurar git sin tener que buscar a cada rato

## Configuraciones del mismo Git

```sh 
git config --global user.name "Felipe Ríos"
git config --global user.email farios@uc.cl
git config --global core.editor "code --wait"
```

Esto para configurar con Visual Code. En ambientes más simples, utiliza nano (Linux)

## Llave para clonar por SSH
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
Settings->(Scurity)Desploy Keys.

Por último se debe inicializar el agente con 
```sh
eval "$(ssh-agent -s)"
```

