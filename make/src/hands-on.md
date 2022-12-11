# Ejercicios

## Ejercicio 1 - contenedor alpine

Hay contenedores, como el de alpine, en el que las siguiente órdenes no funcionan, porque al hacer el run, inmediatamente despúes el contenedor se para:

- `$ docker pull alpine` descarga la imagen
- `$ docker run alpine` crea el contenedor
- `$ docker exec -ti <alpine container>` entra dentro del contenedor

Ordenes que si que funcionan en este caso para entrar dentro del contenedor:

- `$ docker pull alpine` descarga la imagen
- `$ docker run -ti alpine` crea el contenedor y entramos en él

## Ejercicio 2 - contenedor microservicio rest

Sintaxis:

- `$ docker run -p puerto:puerto --name <establecer nombre contenedor> <imagen>`

Ejemplo práctico:

- `$ docker run -p 8080:2222 --name juan_microservicios juan22222/microservicios`


## Ejercicio 3 - contenedor mysql

Crear un contenedor:

- `$ sudo docker run --name <establecer nombre contenedor> -e MYSQL_ROOT_PASSWORD=root -d mysql:latest` descarga la imagen (si no existe) y crea el contenedor
    - `-e` introduce una variable de entorno
    - Se establece `root` como contraseña (el usuario también es root)
    - `latest` coge la última version (comprobarlo)
- `$ sudo docker run --name test-mysql -e MYSQL_ROOT_PASSWORD=root -d mysql` también sirve
- `$ sudo docker exec -ti <container> bash` entra dentro del contenedor

NOTA. En caso de que no usemos el usuario root, hay que especificar nuevo usuario con las variables de entorno `MYSQL_USER`, `MYSQL_PASSWORD`, más información en [https://hub.docker.com/_/mysql](https://hub.docker.com/_/mysql).

Entrar en el contenedor y crear una base de datos:

- `SHOW DATABASES;` indica las bases de datos que existen
- `USE bbdd;` selecciona la base de datos bbdd
- `SHOW TABLES;` muestras las tablas de la base de datos seleccionada
- `CREATE TABLE despliegues (ip VARCHAR(45)PRIMARY KEY, id_imagen VARCHAR(45), replicas INT);`
- `INSERT INTO despliegues VALUES ('192.168.1.43', '555zzz555zzz', 6);`

Crear una imagen a partir del contenedor modificado:

- Parar el contenedor
- `sudo docker commit <container id> <establecer un nombre de imagen>` para crear una imagen propia (local) a partir del contenedor 
- Crear un repositorio en dockerhub de nombre `curso-lucatic-sql-roberto`
- `sudo docker login` para loguearse en la terminal
- `sudo docker tag <imagen> rhonegro/curso-lucatic-sql-roberto:0.0.1` para crear la imagen
- `sudo docker tag <imagen> rhonegro/<image_name>:<tag_name>` sigue la convención de nombres
- `docker push rhonegro/curso-lucatic-sql-roberto:0.0.1` para subir la imagen al repositorio

Volver a repetir los pasos anteriores:

- Puedo eliminar el contendor pero no puedo eliminar la imagen de mysql (porque existe mi imagen, que depende de ella)
- Crear otro contenedor a partir de la imagen de `mysql`, crear bbdd y parar contenedor.
- Parar el contenedor
- `sudo docker commit <container id> <establecer un nombre de imagen>` para crear una imagen propia a partir del contenedor GUARDA IMAGEN PRIMITIVA (hay que meter un volumen para que funcione)
- Crear un repositorio en dockerhub de nombre `curso-lucatic-sql-roberto`
- `sudo docker login` para loguearse en la terminal
- `sudo docker tag <imagen> rhonegro/curso-lucatic-sql-roberto:0.0.1` para crear la imagen
- `sudo docker push rhonegro/curso-lucatic-sql-roberto:0.0.1` para subir la imagen al repositorio

Borrar un repositorio local y descargar el que he borrado desde dockerhub:

- Borrar imagen (tuvo que ser forzosa)
- `sudo docker run --name mysql-test-03 -e MYSQL_ROOT_PASSWORD=root -d rhonegro/curso-lucatic-sql-roberto:0.0.1` decarga imagen y crea contenedor

EN EL CURSO, NO NOS HA FUNCIONADO

- La imagen propia creada que hemos subido a dockerhub no ha subido las modificaciones de las bases de datos. Al hacer el commit local, coge la imagen original

# Crear una imagen docker

We could create our own Docker image in two ways:

## Using Dockerfile to Create an Image

A Dockerfile is a simple text document that contains a series of commands that Docker uses to build an image. Several commands supported in Dockerfile are FROM, CMD, ENTRYPOINT, VOLUME, ENV, and more. A simple Dockerfile looks as follows:

```bash
FROM busybox:latest
CMD ["date"]
```

Note: An important point to remember is that this file should be named as Dockerfile.

The `docker build` command builds an image from a Dockerfile. To build the image from our above Dockerfile, use this command:

```bash
$ docker build -t example_image .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM busybox:latest
---> db8ee88ad75f
Step 2/2 : CMD ["date"]
---> Using cache
---> b89335fdacdf
Successfully built b89335fdacdf
Successfully tagged example-image:latest
```

Now, let’s run a container based on our image. You will find that it will print out the date as shown below:

```bash
$ docker run -it --name example_app example_image
Sun Jul 21 20:52:20 UTC 2019
```

## Create an Image from a Container

Another way to create an image is by pulling a Docker image, creating a container from it, and then modifying or making changes in it like installing our app in that container. Now, using the `docker commit` command, we can create a Docker image from the container.

Let's look at an example of how to create a Docker image from our example_app container:

```bash
$ docker ps -a
CONTAINER ID     IMAGE      COMMAND    CREATED               STATUS              NAMES
c3df2dd33276  example-image "date"  5 seconds ago   Exited (0) 4 seconds ago  example_app

$ docker commit example_app example_image2:latest
sha256:7b48e8355aa7a7ea32d554f26d0bd21f4d069d8526c68f1d098acac9111a9adf
```

# Subir imagen a dockerhub

To be able to publish our Docker images to Docker Hub, there are some steps that we need to follow:

## Step 1: Sign Up for Docker Hub

Before we can push our image to Docker Hub, we will first need to have an account on Docker Hub. Create an account by visiting this link. The signup process is relatively simple.

## Step 2: Create a Repository on Docker Hub
For uploading our image to Docker Hub, we first need to create a repository. To create a repo:

- Sign in to Docker Hub
- Click on ‘Create Repository’ on the Docker Hub welcome page:
- Fill in the repository name as `example-image`, the Docker image that we created earlier using Dockerfile. Also, describe your repo like "My First Repository". Finally, click on the create button. Refer to the below screenshot:

## Step 3: Push Image to Docker Hub

Now we will push our built image to the Docker Hub registry:

1. Log into the Docker public registry from your local machine terminal using Docker CLI:

`$ docker login`

2. Tag the image

This is a crucial step that is required before we can upload our image to the repository. As we discussed earlier, Docker follows the naming convention to identify unofficial images. What we are creating is an unofficial image. Hence, it should follow that syntax. According to that naming convention, the unofficial image name should be named as follows: `<username>/<image_name>:<tag_name>`. In my case, I need to rename it as gauravvv/example_image:latest

`$ docker tag example_image:latest gauravvv/example_image:latest`

3. Publish the image

`$ docker push gauravvv/example_image:latest`

Upload your tagged image to the repository using the docker push command. Once complete, you can see the image there on Docker Hub. That's it; you have successfully published your Docker image. If you want to test out your image, use the below command and launch a container from it:

```bash
$ docker pull gauravvv/example_image:latest
$ docker run -it gauravvv/example_image:latest
```

# Dockerfile

Ejemplo de un Dockerfile:

```bash
# imagen base
FROM centos

# RUn ejecuta comandos
# ADD es para añadir ficheros
RUN yum install -y java-11-openjdk-devel
VOLUME /tmp
ADD prueba-0.0.1-SNAPSHOT.jar app.jar

# RUN y ENTRYPOINT ejecutan cuando arranca el contenedor
# EXPOSE es para indicar el puerto por el que escucha
RUN sh -c 'touch /app.jar' 
EXPOSE 9090 puerto por el que escucha
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

## Ejercicio 1: mysql

Enlace web: https://betterprogramming.pub/customize-your-mysql-database-in-docker-723ffd59d8fb

### Paso 1: Preparación de archivos

Directorio principal (Ubuntu): `/home/roberto/docker_lucatic`

Archivo `./Dockerfile`:

```bash
# la imagen base es mysql
# MYSQL_DATABASE es una variable de entorno
# creamos la bbdd biblioteca
FROM mysql
ENV MYSQL_DATABASE biblioteca

# copiar un script al contenedor
# carpeta origen --> carpeta destino contenedor
COPY ./sql-script/ /docker-entrypoint-initdb.d/
```

Archivo `/sql-script/createtable.sql`:

```sql
USE biblioteca;
CREATE TABLE libros(id_libro INT PRIMARY KEY, autor VARCHAR(45));

INSERT INTO libros VALUES (1, 'marta');
```

Archivo alternativo (no probado):

```sql
CREATE TABLE biblioteca.libros(id_libro INT PRIMARY KEY, autor VARCHAR(45));

INSERT INTO biblioteca.libros VALUES (1, 'marta');
```

### Paso 2: Ejecución de comandos

1. Crear la imagen

Ejecutar desde el directorio del Dockerfile:

`$ sudo docker build -t mysql-dockerfile-image-2 .`

el espacio-punto es necesario.

2. Crear el contenedor

```bash
$ sudo docker run -d --name prueba-mysql -e MYSQL_ROOT_PASSWORD=1234 <imagen>
```

3. Entrar en el contenedor, entrar en mysql y comprobar que se ha creado la tabla.

```bash
$ sudo docker exec -ti <contenedor> bash
```

Dentro del contenedor:

```
/# mysql -u root -p
```

```sql
mysql> show databases;
mysql> use biblioteca;
mysql> show tables;
mysql> select * from libros;
```

# Docker-compose

## Ejecutar junto a un Dockerfile

Archivo `Dockerfile`:

```bash
FROM rhonegro/prueba-mysql-image:0.0.1

# copiar un script al contenedor
# carpeta origen --> carpeta destino contenedor
COPY ./sql-script/ /docker-entrypoint-initdb.d/
```

Archivo `docker-compose.yml`:

```bash
# file: docker-compose.yml
# ejecutar en terminal: docker-compose up -d
# Use root/root as user/password credentials
version: '2'
services:
  # 'testsql' será parte del nombre del contenedor
  testsql:
    # le indico donde está el Dockerfile (directorio actual)
    build: .
    ports:
      - 3306:3306
    # variables de entorno
    environment:
      MYSQL_ROOT_PASSWORD: root

```

## Ejecutar solo

Archivo `docker-compose.yml`:

```bash
# file: docker-compose.yml
# ejecutar en terminal: docker-compose up -d
# Use root/root as user/password credentials
version: '2'
services:
  # 'testsql' será parte del nombre del contenedor
  testsql:
    # le indico la imagen
    image: mysql
    # cambiamos el primer puerto si entra en conflicto con o
    ports:
      - 3307:3306
    # variables de entorno
    environment:
      MYSQL_ROOT_PASSWORD: root
```

Comandos útiles:

- `sudo docker-compose up -d` crear los contenedores (y los arranca en segundo plano)
- `sudo docker-compose down` elimina los contenedores generados con el docker-compose

# Pull indicando puertos

`sudo docker run -p 3306:3306 --name test-ports -e MYSQL_ROOT_PASSWORD=root -d rhonegro/prueba-mysql-image:0.0.1`

# 01/04/2022

## Ejercicio 1

Crear y entrar en un contenedor ubuntu utilizando un Dockerfile. En el Dockerfile hay que crear una carpeta de ubuntu con el nombre roberto (en el directorio raíz)

Dockerfile:

```bash
# la imagen base es ubuntu
FROM ubuntu

# crear una carpeta con mi nombre
RUN mkdir roberto
```

NOTA. Para entrar dentro de algunos contenedores, solo se puede hacer cuando hacemos el run de la imagen.

Comandos en terminal:

- `sudo docker build -t ubuntu-image .`
- `sudo docker run -ti ubuntu-image /bin/bash`

## Ejercicio 2

Idem anterior creando y entrando en dos contenedores ubuntu. Comprobar que están en la misma red con el comando de docker. 

Dentro de un contenedor ubuntu, detectar en la red el otro contenedor con los siguientes comandos:

```bash
apt update
apt install iputils-ping 
ping <ip del otro contenedor de la red>
```

## Ejercicio 3

Se pouede entrar en ubuntu de la menera normal

`sudo docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <ID CONTENEDOR>` para que me de la id del contendor
crear alias --> inspect_ip

# Redes


# 04/04/2022

# Docker y servicio rest

- Creo aplicación
- Dockerizo la aplicación --> imágenes
- Arranco contendor con `sudo docker run -p <establezco puerto>:<puerto> -d <imagen>`

# 05/04/2022

# Docker y servicio rest

## Construcción del archivo .jar desde Eclipse


## Construcción del archivo .jar desde CLI





## Construcción del archivo .jar indicando un nombre de archivo personalizado

Para indicar el nombre al archivo .jar que creamos desde Eclipse, hay que añadir la línea `<finalName>microservicio_deployment</finalName>` en el archivo pom. Se indica en el siguiente fragmento de código:

```pom
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
		<finalName>microservicio_deployment</finalName>
	</build>
```


## Dockerizar el servicio rest - Opción 1

Pasos:

1. Lanzar contenedores con las bbdd de productivo, evolutivo, desarrollo
2. Cambiar properties del servicio rest en Eclipse
  - Se pone la ip del contenedor de la bbdd de desarrollo: `172.25.0.4`
  - Se pone el puerto de mysql: `3306`

```aplication.properties
server.port=3320
spring.datasource.url= jdbc:mysql://172.25.0.4:3306/curso?allowPublicKeyRetrieval=true&useSSL=false
```

3. Generar el .jar desde eclipse
4. Construir la imagen

```bash
sudo docker build -t webapp .
```

5. Levantar el contenedor

```bash
sudo docker run -p 3320:3320 --name webapp_1 webapp
```

## Dockerizar el servicio rest - Opción 2

Pasos:

1. Lanzar contenedores con las bbdd de productivo, evolutivo, desarrollo
2. Cambiar properties del servicio rest en Eclipse
  - Se pone la ip indicada en el docker-compose: `192.168.0.3`
  - Se pone el puerto de mysql: `3306`

```aplication.properties
server.port=3320
spring.datasource.url= jdbc:mysql://192.168.0.3:3306/curso?allowPublicKeyRetrieval=true&useSSL=false
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name =com.mysql.cj.jdbc.Driver
```

3. Generar el .jar desde eclipse
4. Construir la imagen

```bash
sudo docker build -t webapp .
```

5. Levantar el contenedor

```bash
sudo docker run -p 3320:3320 --name webapp_1 webapp
```

```
# file: docker-compose.yml
# ejecutar en terminal: sudo docker-compose up -d
version: '3'

services:

  rest:
    build: .
    ports:
      - "3320:3320"
    environment:
      MYSQL_ROOT_PASSWORD: root
    restart: on-failure
    command: ["sleep", "infinity"]
    networks:
      lan:
        ipv4_address: "192.168.0.2"

  bbdd:
    image: victoria1991/victoriasql:v1
    ports:
      - "3307:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
    networks:
      lan:
        ipv4_address: "192.168.0.3"
        
networks:
    lan:
       driver: bridge
       driver_opts:
           parent: eth0
       ipam:
           driver: default
           config:
               - subnet: "192.168.0.0/24"
                 gateway: "192.168.0.1"
```


# 06/04/2022


# Otro método apra dockerizar 

NOTA. Este método lo van a aliminar


```properties
server.port=9999
spring.datasource.url= jdbc:mysql://alias:3306/curso?allowPublicKeyRetrieval=true&useSSL=false
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name =com.mysql.cj.jdbc.Driver
```


Comandos:

- `sudo docker build -t <imagen de mysql>`

```bash
$ sudo docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root <imagen_1>`
$ sudo docker run -p 9999:9999 --name <establer nombre contenedor> --link <contenedor>:alias <imagen>
```

Mi ejemplo:

```properties
server.port=9999
spring.datasource.url= jdbc:mysql://prueba:3306/curso?allowPublicKeyRetrieval=true&useSSL=false
spring.datasource.username=root
spring.datasource.password=root
spring.dat
```

- `sudo docker build -t prueba .`
- `docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root a347fc9dd762`
- `sudo docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root a347fc9dd762`
- `sudo docker run -p 9999:9999 --name rest --link mysql:prueba prueba`



# Otro método apra dockerizar 

En el docker-compose
