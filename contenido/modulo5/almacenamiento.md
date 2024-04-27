# Almacenamiento compartido entre los contenedores de un Pod

Aunque, como veremos en el próximo ejemplo, cada contenedor puede tener su medio de almacenamiento independiente. Al crear un Pod podemos indicar un punto de montaje, que será compartido entre todos los contenedores del Pod. 

El punto de montaje se puede indicar usando volúmenes o usando bind mount.

## Ejemplo de almacenamiento compartido en un Pod

En este ejemplo vamos a tener el siguiente escenario:

* Creamos un Pod, llamado `pod5` donde vamos a mapear el puerto 8082 al puerto 80 del Pod, y vamos a indicar un punto de montaje con un volumen.
* El contenedor `web` se crea a partir de la imagen nginx, es el contenedor principal, encargado de servir la web. En este contenedor montamos el volumen en su *DocumentRoot* (`/usr/share/nginx/html`). Este servidor web va a servir el fichero `index.html` que está modificando el otro contenedor.
* El contenedor `sidecar` es el auxiliar. En este caso, cada segundo, va a modificar el fichero `index.html` que sirve el contenedor principal.

Para ello primero creamos el Pod, en el este caso usando un volumen:

```
$ podman pod create --name pod5 -p 8082:80 -v vol1:/usr/share/nginx/html
```

Utilizando bind mount, quedaría de la siguiente forma. Utilizaremos la opción `:z` si estamos usando SELinux:

```
$ podman pod create --name pod5 -p 8082:80 -v ${PWD}/directorio_compartido:/usr/share/nginx/html:z
```

A continuación añadimos los contenedores:

```
$ podman run --pod pod5 -d --name web docker.io/nginx
$ podman run --pod pod5 -d --name sidecar docker.io/debian bash -c "while true; do date >> /usr/share/nginx/html/index.html;sleep 1;done"
```

Podemos comprobar que los dos contenedores tienen el volumen montado en el directorio indicado:

```
$ podman inspect --format='{{json .Mounts}}' web
[{"Type":"volume","Name":"vol1","Source":"/var/lib/containers/storage/volumes/vol1/_data","Destination":"/usr/share/nginx/html","Driver":"local","Mode":"","Options":["nosuid","nodev","rbind"],"RW":true,"Propagation":"rprivate"}]

$ podman inspect --format='{{json .Mounts}}' sidecar
[{"Type":"volume","Name":"vol1","Source":"/var/lib/containers/storage/volumes/vol1/_data","Destination":"/usr/share/nginx/html","Driver":"local","Mode":"","Options":["nosuid","nodev","rbind"],"RW":true,"Propagation":"rprivate"}]
```

Por último, podemos acceder al puerto 8082 de la dirección IP del host, para acceder al servicio web:

```
$ curl http://localhost:8082
Thu Apr  4 18:03:12 UTC 2024
Thu Apr  4 18:04:44 UTC 2024
Thu Apr  4 18:04:45 UTC 2024
...
```
