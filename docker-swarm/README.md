# Docker-Swarm 🐳

Docker-Swarm es una solución para balancear la carga automáticamente. La solucón de clustering nativa que ofrece Docker para que nosotros, administradores y desarrolladores solo vean un Docker deamos.

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

