# Docker Swarm 🐳

Docker Swarm es una solución para balancear la carga automáticamente. La solucón de clustering nativa que ofrece Docker para que nosotros, administradores y desarrolladores solo vean un Docker deamon.

Para ejecutar aplicaciones productivas, esta aplicación tiene que estar lista para poder servir a los usuarios en todo momento, a pesar de situaciones catastroficas o de alta demanda.

## Escalabilidad

La capacidad de aumentar la potencia de computo a medida que la demanda avanza.

### Escalabilidad vertical

Más hardware, donde sí hay límite físico.

### Escalabilidad horizontal

Distribuir esta carga entre diferentes ordenadores. Esta es la más usada.

### Disponibilidad

Es la capacidad de una aplicación o servicio para poder estar siempre disponible. Prevée problemas con servidores que fallen, etc...

La escabilidad horizontal y la disponibilidad van de la mano.

## Arquitectura

Docker Swarm plantea un esquema de dos tipos de máquinas:

### Manager Nodes

Son aquellos que se encargan de Administrar el Swarm el clúster de máquinas en si, donde vigila donde hay recursos para garantizar la disponibilidad.

Si hay alguna catástrofe en un servidor, este se tiene que encargar de volver a distribuir la carga entre los Worker Nodes restantes.

### Worker Nodes

Són más que los managers, aquí se ejecutarán los contenedores y son el núcleo de la aplicación productiva. Los únicos requisitos para configurarlos serán:

* Tener el Docker Deamon instalado, idealmente con la misma versión.
* Deben estar todos en la misma red.

Debemos saber si nuestras aplicaciones están listas para correr en Docker Swarm.

## Requisitos de la aplicación

Los requisitos para poder ejecutar nuestras aplicaciones deberian de ser:

* Codebase: Tú aplicación tiene que estár versionado. Por ejemplo. Git, SVM, etc...

* Dependencias: Tienen que estar declaradas y empaquetadas.

* Configuración: Mismo código para todas en el mismo lugar.

* Backing services: Deben estar apuntando a servicios. No tiene que estar en la misma aplicación.

* Build, Release, Run: Deben estar bien definidas en el workflow.

* Processes: La aplicación se tiene que ejecutar como un proceso stayless, no puede esperar cierto estado, todo se debe de realizar de forma atómica.

* Port binding: Las aplicaciones se tienen que exponer directamente a través de un puerto en la máquina que están corriendo, sin intermediarlos.

* Concurrencia: La aplicación tiene la capacidad de correr diferentes instancias en paralelo.

* Disposability: La aplicación tiene que estar diseñada para ser facilmente destruible.

* Dev/Prod parity: Lograr que nuestro entorno de desarrollo sea lo más parecido a producción.

* Logs: Todos los logs deben tratarso como un flujo de device. Tiene que cumplir el Standard Output.

* Admin processes: La aplicación debe poder ejecutar procesos como independientes.

## Comandos iniciales

Para poder iniciar el Swarm de Docker, donde iniciará nuestro nodo como un Manager:

```bash
docker swarm init
```

Para poder añadir un Worker Node, deberemos ejecutar:

```bash
docker swarm join --token <token> <worker_ip>:<worker_port>

# Ejemplo

docker swarm join --token <token> 127.0.0.1:2377
```

Para añadir Manager Nodes, deberemos ejecutar:

```bash
docker swarm join-token manager

# Este comando nos mostrará el comando a ejecutar, con el tocken correspondiente. 

docker swarm join --token <token> <manager_ip>:<manager_port>
```

Para poder listar nuestros nodos, donde nos mostrará el ID de este nodo, hostname, estado, disponibilidad, tipo de manager y la versión del Docker deamon; tendremos que ejecutar:

```bash
docker node ls
```

Para poder inspeccionar la configuración del nodo, deberemos de ejecutar:

```bash
docker node inspect <ID/self>

docker node inspect --pretty self

# En este caso, deberemos de poner el ID del nodo. Si por ejemplo ya estamos en una conexión SSH dentro del nodo a administrar, podemos especificar *self* para indicar el mismo nodo

# La flag *--pretty* se utiliza para imprimir la información estructurada. Esta opción puede llegar a ocultar información.
```

Para abandonar el Swarm

```bash
docker swarm leave

# Si aparece el siguiente error, significa que estamos intentando salir como nodo Manager.

# Error response from daemon: You are attempting to leave the swarm on a node that is participating as a manager. Removing the last manager erases all current state of the swarm. Use `--force` to ignore this message.

# Para forzar la salida de este swarm, añadiremos el flag *--force*.

docker swarm leave -f

# Node left the swarm.
```

Si un Worker Node se va de un Swarm, los Manager Node los pondrán a correr en otros Worker Node. Si se va un Manager Node, hay riesgo de perder y sea irrecuperable el propio Swarm.

Para poder ver el estado de Swarm, si está inactivo o no, podemos ejecutar el siguiente comando:

```bash
docker info | grep Swarm

#  Swarm: active
```

## Fundamentos de Docker Swarm

### Servicios

En Swarm no se corren contenedores manualmente, se declaran los servicios que se desean correr.

Para crear un servicio, se deberá de ejecutar de la siguiente manera el la terminal

```bash
docker service create --name <name> <image> <command>

# Ejemplo

docker service create --name pinger alpine ping www.google.com

## Alpine es una distribución Linux muy ligera orientada a la seguridad.
```

Para poder ver los servicios que están corriendo, debemos ejecutar este otro comando:

```bash
docker service ls
```

#### Ciclo de vida

Desde el cliente, *docker service create* le envía al Manager Node el servicio: se crea, verifica cuantas tareas tendrá, se le otorga una dirección IP virtual y asigna la tarea a los nodos; esta información es recibida por los Worker Node, que es quien prepara la tarea y seguidamente ejecuta los contenedores.
Para poder ver estos servicios, se deberá de ejecutar:

```bash
docker service ps <service_name>

# Ejemplo

docker service ps pinger
```

Para poder inspeccionar el servicio, podemos ejecutar:

```bash
docker service inspect <service_name>

# Ejemplo

docker service inspect pinger
```

Para ver los logs de un servicio, se puede ejecutar:

```bash
docker service logs <service_name> -f

# Ejemplo

docker service logs pinger -f

## La opción *-f* se utiliza para hacer *follow* a dichos logs
```

Para poder borrar el servicio, se utiliza:

```bash
docker service rm <service_name>

# Ejemplo

docker service rm pinger

## Este comando elimina el servicio, pero el contenedor lo deja ejecutando durante unos segundos más.
```