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

Para borrar todos los contenedores, aunque no estén parados, podemos ejecutar este otro:

```bash
docker rm -f $(docker ps -aq)
```

Para poder limpiar el entorno de Docker

```bash
docker system prune
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

Para poder limitar algún recurso a los contenedores, por ejemplo de memoria con la flag *--memory*, tendremos que ponerlo de la siguiente manera:

```bash
docker run -d --name <container_name> --memory <quantity> <image_name>

# Ejemplo

docker run -d --name app --memory 1g platziapp
```

Donde, para ver el consumo de este, se ejecutará:

```bash
docker stats
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

docker run -d --name db -v ".\mongodata":/data/db mongo
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

Para poder limpiar todos los volumenes que se estén usando, usaremos el siguiente comando:

```bash
docker volume prune
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

#### EntyPoint vs CMD

El comando ENTRYPOINT es un comando que se ejecutará siempre por defecto; en CMD, como se puede superponer un comando se puede usar como parámetro por defecto.

```dockerfile
FROM ubuntu:trusty
ENTRYPOINT ["/bin/ping", "-c", "3"]
CMD ["localhost"]
```

#### .dockerignore

El fichero *.dockerignore* nos permite facilmente decidir que archivos queremos que esté en nuestra imagen y cuales no. Funciona igual que *.gitignore*. Un ejemplo de esto puede ser el siguiente archivo:

```text
*.log
.dockerignore
.git
.gitignore
build/*
Dockerfile
node_modules
npm-debug.log*
README.md
```

#### Build multi-stage

Los multi-stage build en docker, nos permite crear imágenes de build y test, previas a la imágen que usaremos para la publicación de nuestro servicio en producción. También es util para reutilizar las capas usando la caché y optimizar la velocidad de build.

```dockerfile
FROM node:12 as builder

COPY ["package.json", "package-lock.json", "/usr/src/"]

WORKDIR /usr/src

RUN npm install --only=production

COPY [".", "/usr/src/"]

RUN npm install --only=development

RUN npm run test


# Productive image
FROM node:12

COPY ["package.json", "package-lock.json", "/usr/src/"]

WORKDIR /usr/src

RUN npm install --only=production

COPY --from=builder ["/usr/src/index.js", "/usr/src/"]

EXPOSE 3000

CMD ["node", "index.js"]

```

### Subir imagen a Dockerhub

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

Para poder eliminar todas las redes que no se estén usando, podemos ejecutar

```bash
docker network prune
```

## Docker Compose

Docker Compose es una herramienta para poder levantar una aplicación con los parámetros ya especificados. Este será un fichero llamado *docker-compose.yml* como el que nos encontramos a continuación:

```yml
docker-compose.yml
version: "3.8" # Obligatorio, es la versión del compose-file.

services: # Obligatorio, Se añaden todos los servicios de nuestra aplicación para poder ejecutarse correctamente.
  app: # Servicio APP
    image: platziapp # Especifica la imagen que se usa
    environment: # Especifica las variables de entorno
      MONGO_URL: "mongodb://db:27017/test"
    depends_on: # Este contenedor, depende DB. Si este no se puede levantar, el principal no se levanta
      - db
    ports: # Expsición de los puertos especificados
      - "3000:3000"

  db: # Servicio DB
    image: mongo # Especifica la imagen que se usa
```

Para poder hacer un build de un Dockerfile directamente desde el Docker Compose, deberemos de cambiar la opción *image* por *build* y especificar el directorio donde se encuentra este:

```yml
version: "3.8"

services:
  app:
    build: .
```

Para añadir volumenes, tendremos que añadir a la misma altura que *services* un apartado llamado *volumes*. Como en el siguiente ejemplo.

```yml
    volumes:
      - .:/usr/src # Ruta host:Ruta contenedor
      - /usr/src/node_modules # Rutas a ignorar
```

También tenemos la opción para poder tener un rango de puertos, para poder escalar nuestra aplicación entre diferentes contenedores y no hayan conflictos:

```yml
    ports:
      - "3000-3001:3000"
```

Para ejecutar comandos, tendremos que añadir a la misma altura que *services* un apartado llamado *command*. Como en el siguiente ejemplo.

```yml
    command: npx nodemon -L index.js # Comando que se ejecuta al inicio
```

Solamente en este caso se deberá de usar el comando, donde a parte, necesitaremos tener nuestro Dockerfile:

```bash
docker-compose build <nombre_servicio>

# El nombre del servicio es opcional. Si hacemos un cambio en uno de nuestros contenedores, no hace falta hacerlo en todos; podemos especificar el nombre del servicio del cual queremos hacer un build:
```

Para poder ejecutar el siguiente archivo para crear los contenedores, deberemos de ejecutar el comando:

```bash
docker-compose up -d
```

En el caso que tengamos que escalar esta aplicación para tener dos contenedores de una misma aplicación, podemos añadirle las siguientes *flags*:

```bash
docker-compose up -d --scale app=2

# 0abdef73c48f   platziapp   "docker-entrypoint.s…"   6 minutes ago   Up 5 minutes   0.0.0.0:3001->3000/tcp   docker_app_2
# c8d389f08eb3   platziapp   "docker-entrypoint.s…"   6 minutes ago   Up 6 minutes   0.0.0.0:3000->3000/tcp   docker_app_1
# 9e3dffdcba15   mongo       "docker-entrypoint.s…"   6 minutes ago   Up 6 minutes   27017/tcp                docker_db_1
```

### Override

Podemos crear un fichero Docker Compose llamado *docker-compose.override.yml* para sobreescribir alguna de la configuración del fichero original *docker-compose.yml*. Esto último se puede usar para tener una configuración segura y no guardar los cambios en el repositorio git, donde también se podrán combinar.

Este fichero puede quedar como este:

```yml
docker-compose.override.yml
version: '3.8'

services:
    app:
      build: .
      environment:
        UNA_VAR: "Hola Mundo"
```

### Comandos de Docker Compose

Igual que en Docker, en Docker Compose podemos ver los servicios/contenedores que se están ejecutando:

```bash
docker-compose ps

# NAME                COMMAND                  SERVICE             STATUS              PORTS
# docker-app-1        "docker-entrypoint.s…"   app                 exited (143)
# docker-db-1         "docker-entrypoint.s…"   db                  exited (0)
```

También, este, conecta los diferentes servicios en una red. En este caso la podremos consultar con el comando propio de Docker.

Como en Docker, también podemos ver los logs de nuestros servicios con:

```bash
docker-compose logs <nombre_servicio>

# El nombre del servicio es una flag opcional, donde si lo obviamos nos mostrará el de todos los servicios.
```

Para hacer un *follow* de los logs, podremos añadirle la opción *-f*. Así se irán actualizando cuando salgan de nuevos.

```bash
docker-compose logs -f <nombre_servicio>
```

Para entrar en la Shell de un servicio se tendrá que hacer de la siguiente manera:

```bash
docker-compose exec <servicio> bash
```

Para eliminar todo lo que ha creado anteriormente, se tendrá que ejecutar:

```bash
docker-compose down
```

## Docker in Docker

Con Docker in Docker conseguimos montar y correr un contenedor de docker dentro de un contenedor de docker.
