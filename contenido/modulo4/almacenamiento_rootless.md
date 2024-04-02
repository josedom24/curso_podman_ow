# Trabajando con almacenamiento en contenedores rootless

* La manera de trabajar con el almacenamiento en contenedores rootless es similar a la vista anteriormente con contenedores rootful.
* Cuando trabajamos con volúmenes con un usuario no privilegiado el directorio donde se crean los volúmenes es `~/.local/share/containers/storage/volumes/`.
* De manera similar cuando se utilice un bind mount con un directorio que no es accesible por el contenedor a causa de SELinux, tendremos que usar las opciones adecuadas para configurar el directorio y hacerlo accesible.
* 

## Uso de volúmenes con contenedores rootless con procesos en el contenedor ejecutándose como root

Si creamos un volumen y lo montamos en un contenedor rootless cuyos procesos se están ejecutando con el usuario `root`. Veamos los usuarios propietarios de los ficheros:

```bash
$ podman volume create vol1

$ podman run -dit -v vol1:/mnt --name alpine alpine

$ podman exec alpine ls -ld /mnt
drwxr-xr-x    1 root     root             0 Jan 26 17:53 /mnt
```

Cómo cabría esperar, comprobamos que el directorio que hemos montado pertenece al usuario `root`.

Podemos inspeccionar el volumen y obtenemos el directorio donde se ha creado en el host:


```bash
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

Estamos utilizando un usuario sin privilegio en el host con UID = 1000. Veamos el propietario del directorio donde se almacenan los ficheros del volumen en el host:

```bash
$ id
uid=1000(usuario) gid=1000(usuario) groups=1000(usuario),4(adm),10(wheel),190(systemd-journal) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

$ ls -ld .local/share/containers/storage/volumes/vol1/_data/
drwxr-xr-x. 1 usuario usuario 0 Jan 26 17:53 .local/share/containers/storage/volumes/vol1/_data/
```

Comprobamos que en el host el propietario es nuestro usuario sin privilegios.

Si creamos un fichero en el volumen desde el host o desde el contenedor:

```bash
$ touch .local/share/containers/storage/volumes/vol1/_data/fichero1
$ podman exec alpine touch /mnt/fichero2
```

Comprobamos que dentro del contenedor pertenecen a `root`:

```bash
$ podman exec alpine ash -c "ls -l /mnt"
total 0
-rw-r--r--    1 root     root             0 Apr  2 08:14 fichero1
-rw-r--r--    1 root     root             0 Apr  2 08:15 fichero2
```
Y que en el host pertenecen al usuario `usuario`:

```bash
$ ls -l .local/share/containers/storage/volumes/vol1/_data/
total 0
-rw-r--r--. 1 usuario usuario 0 Apr  2 08:14 fichero1
-rw-r--r--. 1 usuario usuario 0 Apr  2 08:15 fichero2
```

## Uso de bind mount con contenedores rootless con procesos en el contenedor ejecutándose como root

En este caso el uso de bind mount no tiene ninguna dificultad, ya que los usuarios del host y del contenedor están relacionados: el directorio que queremos montar pertenece al usuario `usuario` en el host y pertenece al usuario `root` en el contenedor.

Si tenemos activo SELinux tendremos que usar la opción `:z` o `:Z` según nos interese. El el directorio `web` tenemos un fichero `index.html`:

```bash 
$ ls -l web
total 4
-rw-r--r--. 1 usuario usuario 15 Apr  2 07:53 index.html
```
Creamos el contenedor y comprobamos el propietario del fichero:

```bash
$ podman run -dit -v ${PWD}/web:/mnt:Z --name alpine alpine

$ podman exec -it alpine ls -l /mnt
total 4
-rw-r--r--    1 root     root            15 Apr  2 07:53 index.html
```

En resumen, el uso de volúmenes o bind mount en contenedores rootlees cuando se ejecutan los procesos como `root` es muy similar a hacerlo con contenedores rootful. Simplemente tenemos que tener en cuenta que el directorio donde se crean los volúmenes se encuentra en el home del usuario: `~/.local/share/containers/storage/volumes/`.

##  Uso de volúmenes con contenedores rootless con procesos en el contenedor ejecutándose con usuario sin privilegios

