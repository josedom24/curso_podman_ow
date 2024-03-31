# Trabajando con volúmenes en Podman

Este apartado lo vamos a realizar con contenedores rootful, en un apartado concreto estudiaremos el almacenamiento en contenedores rootless.

## Gestionando volúmenes

Algunos comandos útiles para trabajar con volúmenes, son los siguientes.

Podemos crear un volumen indicando su nombre:

```bash
$ sudo podman volume create my-vol
```

Para listar los volúmenes que tenemos creados:

```bash
$ sudo podman volume ls
DRIVER      VOLUME NAME
local       my-vol
```

Para obtener información de un volumen:

```bash
$ sudo podman volume inspect my-vol
[
     {
          "Name": "my-vol",
          "Driver": "local",
          "Mountpoint": "/var/lib/containers/storage/volumes/my-vol/_data",
          "CreatedAt": "2024-03-31T18:14:36.528093953Z",
          "Labels": {},
          "Scope": "local",
          "Options": {},
          "MountCount": 0,
          "NeedsCopyUp": true,
          "NeedsChown": true,
          "LockNumber": 0
     }
]
```

Y para eliminar un volumen, ejecutamos:

```bash
$ sudo podman volume rm my-vol
```

## Creación de contenedores con volúmenes

Lo primero que vamos a hacer es crear un volumen:

```bash
$ sudo podman volume create miweb
miweb
```

A continuación, creamos un contenedor con el volumen asociado, usando el parámetro `--mount`. En este ejemplo vamos a montar nuestro volumen en el directorio *DocumentRoot* del servidor Apache que nos ofrece la imagen `httpd:2.4` que encontramos en el registro Docker Hub (en la documentación de la imagen se nos indica que el directorio *DocumentRoot* es `usr/local/apache2/htdocs`).

```bash
$ sudo podman run -d --name my-apache-app --mount type=volume,src=miweb,dst=/usr/local/apache2/htdocs -p 8080:80 docker.io/httpd:2.4
```

Podemos comprobar en la información del contenedor los puntos de montajes que tiene configurado:

```bash
$ sudo podman inspect --format='{{json .Mounts}}' my-apache-app
[{"Type":"volume","Name":"miweb","Source":"/var/lib/containers/storage/volumes/miweb/_data","Destination":"/usr/local/apache2/htdocs","Driver":"local","Mode":"","Options":["nosuid","nodev","rbind"],"RW":true,"Propagation":"rprivate"}]
```

A continuación, creamos un fichero `index.html` en el directorio donde hemos montado el volumen, por lo tanto esta información no se perderá:

```bash
$ sudo podman exec my-apache-app bash -c 'echo "<h1>Hola</h1>" > /usr/local/apache2/htdocs/index.html'
```

Podemos comprobar el acceso al servidor web usando un navegado web o en este caso usando el comando `curl`:

```bash
$ curl http://localhost:8080
<h1>Hola</h1>
```

A continuación borramos el contenedor:

```bash
$ sudo podman rm -f my-apache-app 
my-apache-app
```

Podemos comprobar que el volumen no se ha borrado, ejecutando `sudo podman volume ls`.

Después de borrar el contenedor, volvemos a crear otro contenedor con el mismo volumen asociado, en esta ocasión vamos a usar el parámetro `-v`:

```bash
$ sudo podman run -d --name my-apache-app -v miweb:/usr/local/apache2/htdocs -p 8080:80 docker.io/httpd:2.4
```

Y podemos comprobar que no no se ha perdido la información (el fichero `index.html`):

```bash
$ curl http://localhost:8080
<h1>Hola</h1>
```

