# Trabajando con volúmenes
Este apartado lo vamos a realizar con contenedores rootful, en un apartado concreto estudiaremos el almacenamiento en contenedores rootless.

## Gestionando volúmenes

Algunos comandos útiles para trabajar con volúmenes, son los siguientes.

Podemos crear un volumen indicando su nombre:

```
$ sudo podman volume create my-vol
```

Para listar los volúmenes que tenemos creados:

```
$ sudo podman volume ls
DRIVER      VOLUME NAME
local       my-vol
```

Para obtener información de un volumen:

```
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

```
$ sudo podman volume rm my-vol
```

## Creación de contenedores con volúmenes

Lo primero que vamos a hacer es crear un volumen:

```
$ sudo podman volume create miweb
miweb
```

A continuación, creamos un contenedor con el volumen asociado, usando el parámetro `--mount`. En este ejemplo vamos a montar nuestro volumen en el directorio *DocumentRoot* del servidor Apache que nos ofrece la imagen `httpd:2.4` que encontramos en el registro Docker Hub (en la documentación de la imagen se nos indica que el directorio *DocumentRoot* es `usr/local/apache2/htdocs`).

```
$ sudo podman run -d --name my-apache-app --mount type=volume,src=miweb,dst=/usr/local/apache2/htdocs -p 8080:80 docker.io/httpd:2.4
```

Podemos comprobar en la información del contenedor los puntos de montajes que tiene configurado:

```
$ sudo podman inspect --format='{{json .Mounts}}' my-apache-app
[{"Type":"volume","Name":"miweb","Source":"/var/lib/containers/storage/volumes/miweb/_data","Destination":"/usr/local/apache2/htdocs","Driver":"local","Mode":"","Options":["nosuid","nodev","rbind"],"RW":true,"Propagation":"rprivate"}]
```

A continuación, creamos un fichero `index.html` en el directorio donde hemos montado el volumen, por lo tanto esta información no se perderá:

```
$ sudo podman exec my-apache-app bash -c 'echo "<h1>Hola</h1>" > /usr/local/apache2/htdocs/index.html'
```

Podemos comprobar el acceso al servidor web usando un navegado web o en este caso usando el comando `curl`:

```
$ curl http://localhost:8080
<h1>Hola</h1>
```

A continuación borramos el contenedor:

```
$ sudo podman rm -f my-apache-app 
my-apache-app
```

Podemos comprobar que el volumen no se ha borrado, ejecutando `sudo podman volume ls`.

Después de borrar el contenedor, volvemos a crear otro contenedor con el mismo volumen asociado, en esta ocasión vamos a usar el parámetro `-v`:

```
$ sudo podman run -d --name my-apache-app -v miweb:/usr/local/apache2/htdocs -p 8080:80 docker.io/httpd:2.4
```

Y podemos comprobar que no no se ha perdido la información (el fichero `index.html`):

```
$ curl http://localhost:8080
<h1>Hola</h1>
```

Como hemos comprobado, cuando trabajamos con volúmenes no tenemos ningún problema aunque trabajemos con sistemas operativos con SELinux activado, ya que el directorio donde se guarda los volúmenes está configurado de forma adecuada en el contexto de seguridad de SELinux y es accesible desde los contenedores.
