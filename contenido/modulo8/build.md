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

* El parámetro de `-t` nos permite nombrar la imagen.
* Indicamos el directorio de contexto, en nuestro caso `.` porque estoy ejecutando esta instrucción dentro del directorio donde está el fichero `Containerfile`.

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
$ podman run -d -p 8081:80 --name webserver1 josedom24/webserver:v1
```

Y acceder con el navegador a nuestra página:

![webserver](img/webserver1.png)

### Uso de la caché en la construcción de imágenes OCI

Como hemos indicado anteriormente, durante la construcción de una imagen OCI, se van guardando en caché las capas intermedias que se van generando. Vamos a ver qué ocurre si volvemos a construir la imagen después de alguna modificación:

Si modificamos el fichero `index.html` y volvemos a construir la imagen:

```
$ podman build -t josedom24/webserver:v1 .
...
```
    
La construcción será muy rápida, ya que las imágenes intermedias que se van generando no se vuelven a generar porque están guardadas en caché. Sólo se ejecuta la instrucción donde copiamos el fichero que hemos modificado.

Cuando empezamos a construir nuestras propias imágenes, nos encontramos que al listar las imágenes aparecen algunas con el nombre y la etiqueta con el valor `<none>`. Estas son imágenes intermedias que se han generado y que no forman parte de ninguna imagen, por lo tanto pueden ocupar espacio en disco que no es necesario. Estas imágenes se llaman "colgadas" (dangling).

```
$ podman images
REPOSITORY                     TAG          IMAGE ID      CREATED         SIZE
localhost/josedom24/webserver  v1           aa1a19a3a7b0  8 seconds ago   193 MB
<none>                         <none>       a6582925a34d  31 seconds ago  193 MB
docker.io/library/debian       stable-slim  c3c8e6f4e51e  6 days ago      77.8 MB
```

Para eliminar estas imágenes podemos ejecutar:

```
$ podman image prune
WARNING! This command removes all dangling images.
Are you sure you want to continue? [y/N]
```

## Versión 2: Desde una imagen con Apache

En este caso el fichero `Containerfile` sería el siguiente:

```
FROM docker.io/httpd:2.4
COPY public_html /usr/local/apache2/htdocs/
EXPOSE 80
```

* No necesitamos instalar nada, ya que la imagen tiene instalado el servidor web. 
* Siguiendo la documentación de la imagen en Docker Hub sabemos que el *DocumentRoot* del servidor web es el directorio `/usr/local/apache2/htdocs/`. 
* No es necesario indicar el `CMD` ya que por defecto el contenedor creado a partir de esta imagen ejecutará el mismo proceso que la imagen base, es decir, la ejecución del servidor web.

De forma similar, crearíamos una imagen y un contenedor:

```
$ podman build -t josedom24/webserver:v2 .
$ podman run -d -p 8082:80 --name websever2 josedom24/webserver:v2
```

## Versión 3: Desde una imagen con nginx

En este caso el fichero `Containerfile` sería:

```
FROM podman.io/nginx:1.24
COPY public_html /usr/share/nginx/html
EXPOSE 80
```

De forma similar, crearíamos una imagen y un contenedor:

```
$ podman build -t josedom24/webserver:v3 .
$ podman run -d -p 8083:80 --name webserver3 josedom24/webserver:v3
```

