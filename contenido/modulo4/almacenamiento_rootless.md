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

El volumen lo crea el usuario del host, pero se monta con el identificador de usuario del contenedor. Por lo tanto del conteendor se puede escribir, pero desde fuera no se pueda escribir.

```bash
[fedora@podman ~]$ podman volume create vol1
vol1
[fedora@podman ~]$ podman run -dit -u 123:123 -v vol1:/test --name alpine alpine
73d54eef873cc17a4c9de6c0ebe5c7090b1c1140917454c5eb1f76eca0d67dd6
[fedora@podman ~]$ podman exec alpine touch /test/fichero1
[fedora@podman ~]$ touch .local/share/containers/storage/volumes/vol1/_data/fichero2
touch: cannot touch '.local/share/containers/storage/volumes/vol1/_data/fichero2': Permission denied
[fedora@podman ~]$ ls -l .local/share/containers/storage/volumes/vol1
total 0
drwxr-xr-x. 1 524410 524410 16 Apr  2 11:30 _data
[fedora@podman ~]$ podman exec -it alpine ls -ld test
drwxr-xr-x    1 ntp      ntp             16 Apr  2 11:30 test
```

##  Uso de bind mount con contenedores rootless con procesos en el contenedor ejecutándose con usuario sin privilegios

Creamos un directorio con un fichero que pertenecen a `usuario`:

```
$ mkdir test
$ touch test/fichero1
```

```
$ podman run -dit -u 123:123 -v ./test:/test:Z --name alpine2 alpine
b995cd087bf9e18d0f3925b896d782489df29c271447654eafb717eaeb94a294
[fedora@podman ~]$ podman exec alpine2 touch /test/fichero1
touch: /test/fichero1: Permission denied
[fedora@podman ~]$ podman stop alpine2
WARN[0010] StopSignal SIGTERM failed to stop container alpine2 in 10 seconds, resorting to SIGKILL 
alpine2
[fedora@podman ~]$ podman rm alpine2
alpine2
[fedora@podman ~]$ podman run -dit -u 123:123 -v ./test:/test:Z,U --name alpine2 alpine
4ced2326f465b0debedf8f722095167553335ac7a2aa6413cd0dfae52cd9231d
[fedora@podman ~]$ podman exec alpine2 touch /test/fichero1
[fedora@podman ~]$ podman rm alpine2
Error: cannot remove container 4ced2326f465b0debedf8f722095167553335ac7a2aa6413cd0dfae52cd9231d as it is running - running or paused containers cannot be removed without force: container state improper
[fedora@podman ~]$ podman rm -f alpine2
WARN[0010] StopSignal SIGTERM failed to stop container alpine2 in 10 seconds, resorting to SIGKILL 
alpine2
[fedora@podman ~]$ podman unshare chown -R 123:123 test
[fedora@podman ~]$ podman run -dit -u 123:123 -v ./test:/test:Z --name alpine2 alpine
64d76500e8b6835bc1370a005baf4743e250b30f82bf09806ee599cf5965a9ab
[fedora@podman ~]$ podman exec alpine2 touch /test/fichero2
``



