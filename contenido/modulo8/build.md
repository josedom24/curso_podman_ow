# Construcción de imágenes con podman build

Veamos como podemos automatizar la creación de imágenes OCI, usando un fichero `Containerfile` y el comando `podman build`. Puedes encontrar los ficheros necesarios en el [Repositorio con el código de los ejemplos](xxx).

En este ejemplo vamos a construir distintas versiones de una imagen cpara servir una página web estática.

## Versión 1: Desde una imagen base

Vamos a crear un directorio (a este directorio se le llama **contexto**) donde vamos a crear un fichero `Containerfile` y un directorio, llamado `public_html` con nuestra página web:

```
$ ls
Containerfile  public_html
```

En este caso vamos a usar una imagen base de un sistema operativo sin ningún servicio. El fichero `Containerfile` será el siguiente:

```
FROM debian:stable-slim
RUN apt-get update && apt-get install -y apache2 && apt-get clean && rm -rf /var/lib/apt/lists/*
WORKDIR /var/www/html/
COPY public_html .
EXPOSE 80
CMD apache2ctl -D FOREGROUND
```

* Al usar una imagen base `debian:stable-slim` tenemos que instalar los paquetes necesarios para tener el servidor web, en este caso Apache. 
* Además de la instalación del servicio hemos borrado todos los paquetes que nos hemos bajado, con esto conseguimos que la capa que va a crear la instrucción `RUN` sea lo más pequeña posible.
* A continuación añadiremos el contenido del directorio `public_html` al directorio `/var/www/html/` del contenedor, donde nos hemos posicionado con la instrucción `WORKDIR`. 
* Declaramos el puerto donde se va a ofrecer el servicio. Esta definición es sólo informativa.
* Finalmente indicamos el comando que se deberá ejecutar al crear un contenedor a partir de esta imagen: iniciamos el servidor web en segundo plano.

Para crear la imagen ejecutamos:

```
$ podman build -t josedom24/webserver:v1 .
```

Comprobamos que la imagen se ha creado:

```
$ podman images
REPOSITORY                     TAG          IMAGE ID      CREATED         SIZE
localhost/josedom24/webserver  v1           0e1f90edc439  2 minutes ago   193 MB
...
```

Podemos ver cómo se ha creado cualquier imagen, usando el comando `podman history`:

```
$ podman history josedom24/webserver:v1 
ID            CREATED        CREATED BY                                     SIZE        COMMENT
056b65848578  3 minutes ago  /bin/sh -c #(nop) CMD apache2ctl -D FOREGR...  0B          FROM f255ae9ed2e3
<missing>     3 minutes ago  /bin/sh -c #(nop) EXPOSE 80                    0B          FROM 056b65848578
<missing>     3 minutes ago  /bin/sh -c #(nop) COPY dir:3fceef1ec8e5fa7...  505kB       FROM d4242e321f5e
185b5558aeb6  3 minutes ago  /bin/sh -c #(nop) WORKDIR /var/www/html/       0B          FROM 185b5558aeb6
<missing>     3 minutes ago  /bin/sh -c apt-get update && apt-get insta...  115MB       FROM docker.io/library/debian:stable-slim
c3c8e6f4e51e  6 days ago     /bin/sh -c #(nop)  CMD [""]                0B          
<missing>     6 days ago     /bin/sh -c #(nop) ADD file:3a18b01a2f69e97...  77.8MB      
```

Y podemos crear un contenedor:

```
$ podman run -d -p 8081:80 --name webserver josedom24/webserver:v1
```

Y acceder con el navegador a nuestra página:

![webserver](img/webserver1.png)


## Versión 2: Desde una imagen con Apache

En este caso el fichero `Dockerfile` sería el siguiente:

```Dockerfile
# syntax=docker/dockerfile:1
FROM httpd:2.4
COPY public_html /usr/local/apache2/htdocs/
EXPOSE 80
```

* No necesitamos instalar nada, ya que la imagen tiene instalado el servidor web. 
* Siguiendo la documentación de la imagen en Docker Hub sabemos que el *DocumentRoot* del servidor web es el directorio `/usr/local/apache2/htdocs/`. 
* No es necesario indicar el `CMD` ya que por defecto el contenedor creado a partir de esta imagen ejecutará el mismo proceso que la imagen base, es decir, la ejecución del servidor web.

De forma similar, crearíamos una imagen y un contenedor:

```
$ docker build -t josedom24/ejemplo1:v2 .
$ docker run -d -p 80:80 --name ejemplo1 josedom24/ejemplo1:v2
```

## Versión 3: Desde una imagen con nginx

En este caso el fichero `Dockerfile` sería:

```Dockerfile
# syntax=docker/dockerfile:1
FROM nginx:1.24
COPY public_html /usr/share/nginx/html
EXPOSE 80
```

De forma similar, crearíamos una imagen y un contenedor:

```
$ docker build -t josedom24/ejemplo1:v3 .
$ docker run -d -p 80:80 --name ejemplo1 josedom24/ejemplo1:v3
```



1. Vamos a crear un directorio (a este directorio se le llama **contexto**) donde vamos a crear un fichero `Containerfile` y un fichero `index.html`:

    ```
    cd build
    ~/build$ ls
    Containerfile  index.html
    ```
    El contenido de `Containerfile` es:

    ```Dockerfile
    FROM debian:stable-slim
    RUN apt-get update  && apt-get install -y  apache2 
    WORKDIR /var/www/html
    COPY index.html .
    CMD apache2ctl -D FOREGROUND
    ```

2. Para crear la imagen uso el comando `podman build`, indicando el nombre de la nueva imagen (opción `-t`) y el directorio **contexto**.

    ```
    $ podman build -t josedom24/myapache2:v2 .
    ...
    ```
    **Nota:** Pongo como directorio el `.` porque estoy ejecutando esta instrucción dentro del directorio donde está el fichero `Dockerfile`.

    Una vez terminado, podremos comprobar que hemos generado una nueva imagen:

    ```
    $ podman images
    REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
    josedom24/myapache2       v2                  3bd28de7ae88        43 seconds ago      195MB
    ...
    ```
3. En este caso al crear el contenedor a partir de esta imagen no hay que indicar el proceso que se va a ejecutar, porque ya se ha indicando en el fichero `Dockerfile`, con el parámetro `CMD`:

```
$ podman run -d -p 8080:80 --name servidor_web josedom24/myapache2:v2 
```            

Si queremos ver los distintos pasos que hemos ejecutado para construir la imagen, podemos ejecutar la siguiente instrucción:

```
$ docker history josedom24/myapache2:v2
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
b4836c1e7b7f   41 seconds ago   CMD ["/bin/sh" "-c" "apache2ctl -D FOREGROUN…   0B        buildkit.dockerfile.v0
<missing>      41 seconds ago   COPY index.html . # buildkit                    22B       buildkit.dockerfile.v0
<missing>      42 seconds ago   WORKDIR /var/www/html                           0B        buildkit.dockerfile.v0
<missing>      44 seconds ago   RUN /bin/sh -c apt-get update  && apt-get in…   131MB     buildkit.dockerfile.v0
<missing>      9 days ago       /bin/sh -c #(nop)  CMD [""]                 0B        
<missing>      9 days ago       /bin/sh -c #(nop) ADD file:17e64d3a682fd256f…   74.8MB
```

Donde vemos los pasos que hemos ejecutado en la construcción de la imagen.

## Uso de la caché en la construcción de imágenes Docker

Como hemos indicado anteriormente, durante la construcción de una imagen Docker, se van guardando en caché las capas intermedias que se van generando. Vamos a ver qué ocurre si volvemos a construir la imagen después de alguna modificación:

1. Si modificamos el fichero `index.html` y volvemos a construir la imagen:

    ```
    $ docker build -t josedom24/myapache2:v3 .
    ...
    ```
    
    La construcción será muy rápida, ya que las imágenes intermedias que se van generando no se vuelven a generar porque están guardadas en caché. Sólo se ejecuta la instrucción donde copiamos el fichero que hemos modificado.

2. Si modificamos por ejemplo la instrucción donde se ejecuta la instalación del servidor web y ponemos por ejemplo:

    ```
    ...
    RUN apt-get update  && apt-get install -y  apache2 git
    ...
    ```


$ podman image prune
WARNING! This command removes all dangling images.
Are you sure you want to continue? [y/N]


$ podman system prune --volumes
