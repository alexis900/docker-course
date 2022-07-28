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