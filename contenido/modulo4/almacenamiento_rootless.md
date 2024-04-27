# Trabajando con almacenamiento en contenedores rootless

* La manera de trabajar con el almacenamiento en contenedores rootless es similar a la vista anteriormente con contenedores rootful.
* Cuando trabajamos con volúmenes con un usuario no privilegiado el directorio donde se crean los volúmenes es `$HOME/.local/share/containers/storage/volumes/`.
* De manera similar cuando se utilice un bind mount con un directorio que no es accesible por el contenedor por la configuración SELinux, tendremos que usar las opciones adecuadas para configurar el directorio y hacerlo accesible.

## Uso de volúmenes con contenedores rootless con procesos en el contenedor ejecutándose como root

Si creamos un volumen y lo montamos en un contenedor rootless cuyos procesos se están ejecutando con el usuario `root`, vamos los usuarios propietarios de los ficheros:

```
$ podman volume create vol1

$ podman run -dit -v vol1:/destino --name alpine1 alpine

$ podman exec alpine1 ls -ld /destino
drwxr-xr-x    1 root     root             0 Jan 26 17:53 /destino
```

Cómo cabría esperar, comprobamos que el directorio que hemos montado pertenece al usuario `root`.
Podemos inspeccionar el volumen y obtenemos el directorio donde se ha creado en el host:

```
$ podman volume inspect vol1
[
     {
          "Name": "vol1",
          "Driver": "local",
          "Mountpoint": "/home/usuario/.local/share/containers/storage/volumes/vol1/_data",
          "CreatedAt": "2024-04-02T08:11:05.344065435Z",
          "Labels": {},
          "Scope": "local",
          "Options": {},
          "MountCount": 0,
          "NeedsCopyUp": true,
          "LockNumber": 0
     }
]
```

Estamos utilizando un usuario sin privilegios en el host con UID = 1000. Veamos el propietario del directorio donde se almacenan los ficheros del volumen en el host:

```
$ id
uid=1000(usuario) gid=1000(usuario) groups=1000(usuario),4(adm),10(wheel),190(systemd-journal) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

$ ls -ld .local/share/containers/storage/volumes/vol1/_data/
drwxr-xr-x. 1 usuario usuario 0 Jan 26 17:53 .local/share/containers/storage/volumes/vol1/_data/
```

Comprobamos que en el host el propietario es nuestro usuario sin privilegios.
Si creamos un fichero en el volumen desde el host o desde el contenedor:

```
$ touch .local/share/containers/storage/volumes/vol1/_data/fichero1
$ podman exec alpine1 touch /destino/fichero2
```

Comprobamos que dentro del contenedor pertenecen a `root`:

```
$ podman exec alpine1 ash -c "ls -l /destino"
total 0
-rw-r--r--    1 root     root             0 Apr  2 08:14 fichero1
-rw-r--r--    1 root     root             0 Apr  2 08:15 fichero2
```
Y que en el host pertenecen al usuario `usuario`:

```
$ ls -l .local/share/containers/storage/volumes/vol1/_data/
total 0
-rw-r--r--. 1 usuario usuario 0 Apr  2 08:14 fichero1
-rw-r--r--. 1 usuario usuario 0 Apr  2 08:15 fichero2
```

## Uso de bind mount con contenedores rootless con procesos en el contenedor ejecutándose como root

En este caso el uso de bind mount no tiene ninguna dificultad, ya que los usuarios del host y del contenedor están relacionados: el directorio que queremos montar pertenece al usuario `usuario` en el host y pertenece al usuario `root` en el contenedor.

Si tenemos activo SELinux tendremos que usar la opción `:z` o `:Z` según nos interese. El el directorio `web` tenemos un fichero `index.html`:

``` 
$ ls -l web
total 4
-rw-r--r--. 1 usuario usuario 15 Apr  2 07:53 index.html
```
Creamos el contenedor y comprobamos el propietario del fichero:

```
$ podman run -dit -v ${PWD}/web:/destino:Z --name alpine2 alpine

$ podman exec -it alpine2 ls -l /destino
total 4
-rw-r--r--    1 root     root            15 Apr  2 07:53 index.html
```

En resumen, el uso de volúmenes o bind mount en contenedores rootless cuando se ejecutan los procesos como `root` es muy similar a hacerlo con contenedores rootful. Simplemente tenemos que tener en cuenta que el directorio donde se crean los volúmenes se encuentra en el home del usuario: `$HOME/.local/share/containers/storage/volumes/`.

##  Uso de volúmenes con contenedores rootless con procesos en el contenedor ejecutándose con usuario sin privilegios

El volumen lo crea el usuario del host, pero dentro del contenedor es propiedad del usuario del contenedor. Por lo tanto desde contenedor se puede escribir, pero desde fuera no se pueda escribir.

```
$ podman volume create vol2

$ podman run -dit -u 123:123 -v vol2:/destino --name alpine3 alpine

$ podman exec alpine3 touch /destino/fichero1

$ touch .local/share/containers/storage/volumes/vol2/_data/fichero2
touch: cannot touch '.local/share/containers/storage/volumes/vol2/_data/fichero2': Permission denied
```

Podemos ver el propietario del directorio: dentro del contenedor pertenece al usuario que hemos indicado, en este caso es `ntp` que tiene UID y GID igual a 123; fuera del contenedor el directorio donde se guarda la información del volumen pertenece al usuario con UID 524410, que corresponde al UID que se ha mapeado fuera del contenedor.

```
$ podman exec -it alpine3 ls -ld destino
drwxr-xr-x    1 ntp      ntp             16 Apr  2 11:30 destino


$ ls -l .local/share/containers/storage/volumes/vol2
total 0
drwxr-xr-x. 1 524410 524410 16 Apr  2 11:30 _data

```

##  Uso de bind mount con contenedores rootless con procesos en el contenedor ejecutándose con usuario sin privilegios

Creamos un directorio con un fichero que pertenecen al usuario sin privilegios, en nuestro caso `usuario`:

```
$ mkdir origen
$ touch origen/fichero1
```

Creamos un contador y montamos el directorio `origen` con la opción `:Z` para configurar de forma adecuada SELinux y sea accesible desde el contenedor. Comprobamos que al pertenecer el directorio `origen` a nuestro usuario `usuario`, el directorio `destino` será propiedad del `root` (mapeo de `usuario`) y por lo tanto el usuario con UID 123 no podrá acceder al directorio:

```
$ podman run -dit -u 123:123 -v ./origen:/destino:Z --name alpine4 alpine

$ podman exec -it alpine4 ls -ld destino
drwxr-xr-x    1 root     root            16 Apr  2 14:28 destino

$ podman exec alpine4 touch /destino/fichero2
touch: /destino/fichero2: Permission denied
```

Para que el usuario con UID 123 pueda acceder al directorio tenemos que asegurarnos que el directorio `destino` le pertenece. Para conseguir esto lo podemos hacer de dos formas distintas:

1. A la hora de montar el directorio utilizar la opción `:U` que cambia el usuario y grupo de forma recursiva al directorio montado dentro del contenedor con el usuario y grupo que se esté ejecutando dentro del contenedor. En nuestro caso, creamos un nuevo contenedor con dicha opción:

     ```
     $ podman run -dit -u 123:123 -v ./origen:/destino:Z,U --name alpine4 alpine

     $ podman exec -it alpine2 ls -ld destino
     drwxr-xr-x    1 ntp      ntp             16 Apr  2 14:28 destino
     
     $ podman exec alpine4 touch /destino/fichero2
     ```

2. Otra opción sería cambiar el propietario del directorio `origen` en el host con el UID y GID del usuario que se ejecuta dentro del contenedor. Para ello es necesario que esa instrucción la ejecutemos en el espacio de nombres de usuario del contenedor, para ello usaremos la instrucción `podman unshare`:

     ```
     $ podman unshare chown -R 123:123 origen
     ```

     Comprobamos que en fuera del contenedor el UID que se asigna es el correspondiente al mapeo de UID realizado:

     ```
     $ ls -ld origen
     drwxr-xr-x. 1 524410 524410 32 Apr  2 14:39 origen
     ```

     Creamos un nuevo contenedor y comprobamos que dentro del contenedor el cambio de propietario se refleja de forma y correcta (pertenece al usuario `ntp:ntp`, que corresponde con el UID y GID 123) y ahora si podemos acceder al directorio:

     ```
     $ podman run -dit -u 123:123 -v ./origen:/destino:Z --name alpine5 alpine

     $ podman exec -it alpine5 ls -ld destino
     drwxr-xr-x    1 ntp      ntp             32 Apr  2 14:39 destino

     $ podman exec -it alpine5 ls -l destino
     total 0
     -rw-r--r--    1 ntp      ntp              0 Apr  2 14:28 fichero1
     -rw-r--r--    1 ntp      ntp              0 Apr  2 14:39 fichero2

     $ podman exec -it alpine5 touch destino/fichero3
     ```



