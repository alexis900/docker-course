# Docker 🐳

Docker es una plataforma que permite *construir*, *ejecutar* y *compartir* aplicaciones mediante contenedores.

El núcleo de docker es el *docker deamon*.

Para poder correr un contenedor se tiene que ejecutar el siguiente comando

```bash
docker run hello-world
```

## Contenedores

Un contenedor, puede ser como una VM liviana. Esto es una agrupación logica de procesos que corren nativamente en la máquina de manera aislada. Piensa que está en un ordenador fisico, excepto si se lo permitimos.

Para mostrar los contenedores que están activos, ejecutamos:

```bash
docker ps
```

Para mostrar los conteneres que tenemos, ejecutamos:

```bash
docker ps -a
```

Para ver la configuración de un contenedor, ejecutamos:

```bash
docker inspect <container-id/name>

```

Para asignar un nombre a un contenedor, ejecuto:

```bash
docker run --name <name> <container>
```

Para renombrar un contededor, ejecutamos:

```bash
docker rename <actual_name> <new_name>
```

Para borrar un contenedor, ejecutamos:

```bash
docker rm <container-id/name>
```

Para borrar todos los contenedores parados, ejecutamos:

```bash
docker container prune
```

### Modo interactivo

Para acceder al modo interactivo de cualquier máquina, haremos la prueba con un sistema Ubuntu:

```bash
docker run -it ubuntu
cat /etc/lsb-release

# DISTRIB_ID=Ubuntu
# DISTRIB_RELEASE=22.04
# DISTRIB_CODENAME=jammy
# DISTRIB_DESCRIPTION="Ubuntu 22.04 LTS"

```

Lo que pasa, si salimos de este contenedor, este se apagará y no podremos acceder más.

### Ciclo de vida de los contenedores

Cada vez que se ejecuta un contenedor, en realidad se ejecuta un proceso de un sistema operativo. Este proceso se llama Main process.
Mientras este Main process esté corriendo, el contenedor estará corriendo.

El siguiente comando utiliza el comando *tail -d /dev/null* para que quede siempre encendido:

```bash
docker run --name alwaysup -d ubuntu tail -f /dev/null
```

Para poder entrar en el contenedor que está corriendo, podemos ejecutar este otro comando:

```bash
docker exec -it alwaysup bash
```

Para poder finalizar el proceso, desde fuera del contenedor, lo primero que tendremos que hacer es saber el Pid del proceso que está corriendo. Para saber este Pid, tendremos que ejecutar el siguiente comando; donde es el mismo para mostrar la configuración de dicho contenedor, pero con un formato especifico:

```bash
 docker inspect --format '{{.State.Pid}}' alwaysup
 # 1491
```

Si hacemos este comando en Windows o en macOS, para intentar finalizar el proceso, nos saldrá un error parecido a este:

```powershell
└─$ kill 1491
Stop-Process: Cannot find a process with the process identifier 1491.
```

En un computador Linux se ejecutaria de la siguiente manera:

```bash
kill -9 1491
```

Para poderlos parar en estos sistemas operativos, se tendrá que usar:

```bash
docker stop <id/nombre>
```

Para poder borrar el contenedor se usara este otro:
```bash
docker rm <id/nombre>
```

### Exponiendo contenedores

Cada contenedor tiene su propia interfaz de red virtual. Para poder exponer exponer un contenedor tendremos que ejecutar el siguiente comando:

``` bash
docker run --name <name> -d -p <puesto_host>:<puerto_contenedor> <imagen>

# Ejemplo

docker run --name proxy -d -p 8080:80 nginx
```

Para poder ver los logs que se muestran en el contenedor, ejecutaremos lo siguiente:

```bash
docker log <imagen>

# Ejemplo

docker log proxy
```

### Manejo de datos

Al borrarse todos los datos de los contenedores, cuando eliminamos un contenedor también se borran estos datos. Para solucionar este problema, lo que tendremos que hacer es *mapear* un directorio de nuestra maquina host a nuestro contenedor para que puedan guardar estos datos. Para mapear este directorio, usaremos el siguiente comando:

```bash

docker run -d --name <nombre_contenedor> -v <punto_montaje_host>:<punto_montaje_contenedor> <imagen>

# Ejemplo

docker run -d --name db -v "C:\Users\aleja\Projects\Docker\docker-data\mongodata":/data/db mongo
```

En este caso será más seguro crear volumenes por un tema de seguridad, para que Docker no pueda acceder directamente a nuestro disco. Para poder crear los volumenes, deberemos de ejecutar el siguiente comando.

```bash
docker volume create <nombre>

# Ejemplo

docker volume create dbdata
```

Para poder listar estos volumenes, tendremos que ejecutar:

```bash
docker volume ls
```

Para poder montar este volumen a un contenedor, deberemos de ejecutar el siguiente comando:

```bash
docker run -d --name <nombre_contenedor> --mount src=<nombre_volumen>,dst=<punt_montaje_contenedor> <imagen>

# Ejemplo

docker run -d --name db --mount src=dbdata,dst=/data/db mongo
```

#### Insertar y extraer archivos de un contenedor

Para poder copiar un archivo entre la maquina anftriona y el conteneror, podemos ejecutar el comando *cp*, para poder copiar dicho archivo en la ruta especificada:

```bash
docker cp <archivo_host> <contenedor>:<ruta_contenedor>

# Ejemplo

docker cp prueba.txt copytest:/testing
```

Para poder hacerlo de la manera inversa, solamente tendremos que cambiar el orden de los factores. En este caso, no es necesario que el contenedor esté corriendo:

```bash
docker cp <contenedor>:<ruta_contenedor> <archivo_host>

# Ejemplo

docker cp copytest:/testing localtesting
```

## Imágenes

Las imágenes son plantillas o moldes que docker utilitza para crear los contenedores. Es una pieza de software empaquetada que tiene todo lo necesario para que este se ejecute correctamente.

Para listar las imagenes instaladas en nuestro equipo, podemos ejecutar el siguiente comando:

```bash
docker image ls
```

Con este comando podemos distinguir más conceptos de las imágenes, como puede ser el **Tag**. El tag, es como la versión de esta imagen que descargamos. Si no le especificamos ninguna Docker interpreta que es *:latest*.

Estas imágenes, por defecto, se descargan de [Docker Hub](https://hub.docker.com)

Para poder bajar una imágen con un tag diferente, necesitaremos utilizar:

```bash
docker pull ubuntu:20.04
```

### Creación de imagenes

Para poder generar una imágen nueva, primero tendremos que crear un fichero Dockerfile que tendrá que contener como mínimo el comando **FROM** y la imagen base que usaremos, tal y como muestra el ejemplo siguiente. También podremos ejecutar comandos como **RUN** para poder ejecutar los comandos correspondientes.

```docker
FROM ubuntu:latest

RUN touch /usr/src/hola-mundo.txt
```

Para poder generar esta, tendremos que ejecutar el siguiente comando:

```bash
docker build -t <nombre>:<tag> <ruta_dockerfile>

#Ejemplo

docker build -t ubuntu:example .
```

#### Subir imagen a Dockerhub

Para poder subir una imagen a Docker Hub, lo primero que tendremos que hacer es tener una cuenta en dicha plataforma, donde nos tendremos que registrar. Una vez registrados, para loguearnos desde nuestra terminal, necesitamos ejecutar:

```bash
docker login
```

Cuando ejecutamos este comando nos pedirá nuestro usuario y contraseña de Docker Hub.

Para poder crear un nuevo tag, para poder publicar correctamente nuestra imagen, ejecutaremos el siguiente comando:

```bash
docker tag <imagen>:<tag> <usuario>/<imagen>:<tag>

# Ejemplo

docker tag ubuntu:example alexis900/ubuntu:example
```

### Sistema de capas

Estas imágenes son un conjunto de capas, donde, a partir del Dockerfile se puede saber como se ha construido esta. Existen diferentes maneras de ver las capas de esta imagen:

* Si la imagen es pública, se puede visitar Dockerhub y buscar la imagen del Dockerfile y buscarla a través del tag.

* También se puede hacer esto mismo a través de la linea de comandos, aunq esta opción nosea la más comoda. Es importante destacar que en esta opción destaca que los cambios se presentan por filas, donde las filas inferiores son los cambios más viejos. El comando que se ejecutará es:

```bash
docker history <nombre_imagen>:<nombre_tag>

# Ejemplo

docker history ubuntu:platzi
```

### Redes

Para poder intercomunicar distintos contenedores, por ejemplo entre un servicio de NodeJS y una base de datos, los tendremos que conectar a redes virtuales. Estas redes virtuales solamente serán visibiles entre los propios contenedores.

Para poder ver todas las redes virtuales que tenemos configuradas, tendremos que ejecutar el siguiente comando:

```bash
docker network ls

# NETWORK ID     NAME      DRIVER    SCOPE
# b4e576729491   bridge    bridge    local # Red bridge por retrocompatibilidad con versiones anteriores
# d685c54f0233   host      host      local # Representación en Docker de la red de mi máquina y la red.
# b8d921e5bfdc   none      null      local # Red especial, para especificar que el contenedor no puede salir a Internet
```

Para poder crear una red nueva, donde usaremos el flag *attachable* para que otros contenedores se conecten a ella, se ejecutará:

```bash
docker network create --attachable <name>

# Ejemplo

docker network create --attachable networkexample
```

Como otros de los comandos de Docker, el comando *docker network* también incluye la opción de *inspect* para poder ver las caracteristicas de esta.

```bash
docker network inspect <name/id>

# Ejemplo

docker network inspect networkexample
```

Para poder conectar un contenedor a una red virtual, usarmos la opción *connect*, como en el siguiente ejemplo:

```bash
docker network connect <network_name/network_id> <container_name/container_id>

# Ejemplo

docker network connect networkexample db
```

En la creación de contenedores, también podremos añadir una variable de entorno con el flag *--env*, por ejemplo en este cado donde intentamos conectar la aplicación principal con la base de datos. También hay que remarcar, docker puede encontrar los contenedores dentro de su red por nombre, por eso se puede poner directamente db, que hace referencia al contenedor donde está alojada nuestra base de datos:

```bash
docker run -d --name app -p 3000:3000 --env MONGO_URL=mongodb://db:27017/test platziapp

# En este caso también tenemos que añadir este contenedor a la red

docker network connect networkexample app
```