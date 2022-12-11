# Introducción

Una `imagen de contenedor` (imagen docker) representa los datos binarios que encapsulan una aplicación y todas sus dependencia de software. Las imágenes de contenedor son paquetes de sofware ejecutable (executable software bundles) que se pueden ejecutar de manera independiente (run standalone) y que hace suposiciones muy bien definidas sobre el entorno de ejecución.

Un `contenedor` es una instancia en ejecución de una imagen. Es importante tener claro su carácter efímero. Están hechos para poder ser destruídos y volver a levantar una réplica procedentes de la misma imagen. Los cambios realizados en un contenedor, por defecto, no persisten. Es decir, si se elimina y se levanta una replica, se pierden los cambios. Se pueden montar `volúmenes` para persistir datos y/o compartir datos entre contenedores.

`Docker` es una tecnología que permite gestionar (crear, iniciar, detener, eliminar) contenedores. `Docker Hub` es el registro de imágenes de la empresa `Docker`
 
Simplicando mucho, podemos ver un contenedor como una pequeña máquina virtual. Los contenedores se utilizan por varias razones, algunas son estandarización, portabilidad, aislamiento y rapidez para desplegar aplicaciones.

Un `Dockerfile` es un archivo de configuración que sirve para construir una imagen a partir de otra imagen base. 

`Dockercompose` es un sistema de orquestación de contenedores muy simple y que no se usa en producción. Utiliza un fichero llamado `docker-compose.yml` que puede utilizarse (o no) junto a un Dockerfile.

`Kubernetes` es el sistema de orquestación más utilizado en producción en 2022. Utiliza `pods` como unidad mínima de despliegue. Un pod es un set de contenedores, el principal y otros de soporte. Típicamente, se crea una imagen de contenedor para una aplicación y se sube a un registro (en producción no se utiliza Docker Hub, sino `Nexux` o `Harbor` entre otros) antes de referenciar la imagen para que la utilice un Pod.

Las `redes` constituyen un elemento muy importante tanto en Docker como en Kubernetes. La red por defecto emnm Docker es `bridge`.

# Comandos para levantar contenedores

## Descargar una imagen

Para descargar una imagen (almacenada en DockerHub)

```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
docker pull <image name>
```

utilizará `latest` si no se especifica el tag.

## Levantar un contenedor

Para levantar un contenedor a partir de una imagen (la descarga de DockerHub si no existe)

```bash
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
docker run -d --name <set container name> <image name>
```

donde `-d` indica modo `detached` true, es decir, el contenedor se ejecute en segundo plano (`background`)

Por defecto, si no se indica `-d`, se ejecuta en modo `foreground`. En este caso, docker run inicia el proceso en el contenedor y fija (attach) la consola a los procesos STDIN, STDOUT, STDERR. 

Por defecto fija STDIN y STDOUT, pero se puede configurar:

- `-a=[]` fija STDIN, STDOUT y/o STDERR, por ejemplo `-a stdout`
- `-t` asigna (allocate) una `pseudo-tty`
- `-i` mantine STDIN abierto aunque no se fije

Para procesos iteractivos, como una shell, se deben utiliza juntos `-i -t` con el objetivo de asignar una `tty` para el proceso del contenedor

- se suele escribir junto `-it`

Ejemplo:

```bash
docker run -it --name test busybox sh
```

## Flag --rm

El flag `--rm` sirve para eliminar el filesystem que por defecto persiste después de su muerte (volumen anónimo). Ayuda a no sobrecargar el disco después de ejecutar contenedores de vida corta. 

```bash
docker run --rm -it python:3 python
```

Los volúmenes anónimos se guardan normalmente en algún lugar de `/var/lib/docker`, comprobarlo con

```bash
docker inspect --type container -f '{{range $i, $v := .Mounts }}{{printf "%v\n" $v}}{{end}}' <container name>
```

## Utilizando variables de entorno

Flag de `docker run` para introducir variables de entorno

- `-e "<env=value>"` 

## Exposición de puertos

Flag de `docker run` para abrir y mapear puertos

- `-p <hostPort>:<containerPort>` expone un puerto del contenedor al host (mapeo)

El puerto expuesto es accesible en el host y los puertos están disponibles para cualquier cliente que pueda llegar al host.

Formato para la exposición de puertos - EXPOSE (incoming ports)

```lang
ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort | containerPort
```

**NOTA.* El número de puerto dentro del contenedor (donde el servicio escucha) no necesariamente coincide (match) con el número de puerto expuesto fuera del contenedor (donde el cliente conecta). Por ejemplo, en el contenedor un servicio HTTP esta escuchando en el puerto 80 (`EXPOSE 80` en el Dockerfile). En tiempo de ejecución, el puerto podría vincularse (bound) con el puerto 42800 en el host. 

Se pueden listar los mapes de los puertos del host y los puertos expuestos con la orden

```bash
docker port CONTAINER [PRIVATE_PORT[/PROTO]]
docker port <container name>
```

# Orquestar contenedores con docker compose

## Comandos para levantar contenedores con docker-compose

Se ejecuta desde la carpeta que contiene el `docker-compose.yml`

```bash
docker-compose up -d
```

donde `-d` es para que el contenedor se ejecute en segundo plano.

El archivo `docker-compose.yml` puede hacer referencia a un Dockerfile o descargar la imagen de un almacen remoto.

## Comandos para gestionar contenedores

## Comandos para manejar imagenes

- `docker images` lista las imágenes
- `docker rmi <imagen>` elimina una imagen
- `docker system prune` elimina imágenes, contenedores, volúmenes y dispositivos de red, que no estén asociadas a ningún contenedor
- `docker system prune -a` elimina todos los contenedores parados y todas las imágenes que no se están usando en ese momento

## Comandos para manejar contenedores

- `docker ps -a` lista los contenedores que tengo
- `docker ps -a -q` lista los ids de los contenedores que tengo
- `docker ps` lista los contenedores que están corriendo
- `docker ps -q` lista los ids de los contenedores que están corriendo
- `docker inspect <container>` da información sobre el contenedor
- `docker start <container>` arranca un contenedor
- `docker stop <container>` para un contenedor
- `docker rm <container>` elimina un contenedor
- `docker rm -f <container>` fuerza la eliminación de un contenedor
- `docker exec -ti <container> bash` entra dentro de un contenedor 

Comando para detener y borrar todos los contenedores:

- `$ docker stop $(sudo docker ps -a -q)`
- `$ docker rm $(sudo docker ps -a -q)`

## Comandos de redes

- `docker network ls` lista las redes que tengo
- `docker network connect <id red> <id contenedor>` conecta un contenedor a una red
- `docker network inspect <id red>` da información sobre la red

## Comandos para manejar volúmenes

- `docker volume create <nombre al volumen>` crea un volumnen
- `docker volume ls` lista los volúmenes creados
- `docker volume inspect <volumen>` inspeciona donde está guardado el volumen
- `docker run -it -v <volumen>:<punto montaje en el contenedor> ubuntu:latest` arrancar ubuntu enlazando con el volumen creado
- `docker volume rm <volumen>` elimina un volumen
- `docker volume prune` elimina todos los volúmenes que no están siendo usados por ningún contenedor

En el curso borré 25 GB de volúmenes.

# Construcción de imágenes con Dockerfile

## Comando para construir una imagen con un Dockerfile

Para construir una imagen a partir de un Dockerfile

```bash
docker build [OPTIONS] PATH | URL | -
docker build -t <nombre para la imagen> <ruta del Dockerfile>
``` 

Si lo ejecutamos de la carperta del Dockerfile se puede pasar un punto `.` para indicar directorio actual.

Para instanciar un contenedor a partir de la imagen creada

```bash
docker run -d --name <nombre para el contenedor> <imagen>
```

## Instrucciones para esccribir un Dockerfile

Sintaxis utilizada en un Dockerfile. El formato es el siguiente

```dockerfile
# Comment
INSTRUCTION arguments
```

### Instrucciones básicas

- `FROM <image:tag>` especifica la imagen base
- `ENV <key> <value>` establece una variable de entorno persistente
- `ARG` define una variable usable en el resto del Dockerfile con la sintaxis `${NOMBRE_DEL_ARG}`


- `COPY` copia archivos y directorios al contenedor
- `ADD` idem anterior y además puede descomprimir archivos `.tar` y `añadir archivos vía URL`
- `WORKDIR` indica el directorio sobre el que se van a aplicar las instrucciones siguientes

### Ejecución de comandos y punto de entrada

El Entrypoint de Docker define el comando a ejecutar por defecto cuando se inicia el contenedor a partir de su imagen.
Docker tiene por defecto el Entrypoint `/bin/sh -c` pero este se puede sobrescribir con la instrucción ENTRYPOINT.

- `ENTRYPOINT` indica el ejecutable que utilizará el contenedor cuando se inicia
- `CMD` indica los argumentos que se pasan al ejecutable del ENTRYPOINT

**¡Inportante!** Cuando iniciamos un contenedor, podemos indicar una lista de argumentos que le queremos proporcionar:

```bash
docker run <image> <arg1> <arg2>
```

Los argumentos que usemos sobrescribirán el valor de `CMD` en caso de haberlo y serán ejecutados junto con el comando del `ENTRYPOINT`.

- `RUN` ejecuta una instrucción y crea una imagen (instalar paquetes en el contenedor)


### Configurar entrypont del contenedor





- `EXPOSE` indica que puerto del contenedor se debe exponer al ejecutarlo (no lo expone directamente)
- `LABEL` para aportar meta-datos a la imagen
- `VOLUME` crea un directorio sobre el que se va a montar un volumen para persistencia de datos
- `USER` establece el usuario que se va a usar cuando se ejecute cualquier operación posterior con RUN, CMD y ENTRYPOINT
- `SHELL` permite especificar el uso de otra terminal como zsh, csh, tcsh, powershell
- `STOPSIGNAL` indica una señal que va a finalizar el contenedor
- `HEALTHCHECK` indica a Docker una manera de testear el contenedor para verificar que sigue funcionando correctamente
- `ONBUILD` cuando la imagen donde se encuentra se use como base de otra imagen, va a actuar de trigger y va a ejecutar el comando que le indiquemos


Ejemplo de un Dockerfile:

```bash
FROM centos
RUN yum install -y java-11-openjdk-devel
VOLUME /tmp
ADD prueba-0.0.1-SNAPSHOT.jar app.jar
COPY ./holamundo.txt /home/Escritorio/Docker/holamundo.txt
RUN sh -c 'touch /app.jar' 
EXPOSE 9090
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
  




