# Ejecución simple de contenedores

En este apartado vamos a crear un contenedor especificando el comando que debe ejecutar a partir de la imagen `ubuntu`.
En este caso vamos a descargar primero la imagen del registro público Docker Hub, y a continuación crearemos el contenedor.

Para descargar la imagen ejecutamos:

```
$ sudo podman pull ubuntu
Resolved "ubuntu" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/ubuntu:latest...
Getting image source signatures
Copying blob bccd10f490ab done   | 
Copying config ca2b0f2696 done   | 
Writing manifest to image destination
ca2b0f26964cf2e80ba3e084d5983dab293fdb87485dc6445f3f7bbfc89d7459
```

Podemos utilizar el nombre dorto de la imagen `ubuntu` porque tenemos creado un alias en el fichero `/etc/containers/registries.conf.d/000-shortnames.conf`. A continuación, creamos y ejecutamos un nuevo contenedor indicando el comando que va a ejecutar:

```
$ sudo podman run ubuntu echo 'Hello world'
Hello world
```

Comprobamos que el contenedor ha ejecutado el comando que hemos indicado y se ha parado:

```
$ sudo podman ps -a
CONTAINER ID  IMAGE                            COMMAND           CREATED        STATUS                    PORTS       NAMES
6e8e46cee350  docker.io/library/ubuntu:latest  echo Hello world  2 seconds ago  Exited (0) 2 seconds ago              eloquent_hodgkin
```

Los contenedores que hemos creado se nombran de manera aleatoria. Podemos cambiar el nombre de cualquier contenedor usando el comando `podman rename`:

```
$ sudo podman rename eloquent_hodgkin contenedor_ubuntu
```

Y comprobamos que el nombre ha cambiado:

```
$ sudo podman ps -a
CONTAINER ID  IMAGE                            COMMAND           CREATED             STATUS                         PORTS       NAMES
6e8e46cee350  docker.io/library/ubuntu:latest  echo Hello world  About a minute ago  Exited (0) About a minute ago              contenedor_ubuntu
```

## ¿Qué ocurre cuando creamos un contenedor?

Para terminar podemos ver las distintas etapas por las que pasa la creación de un contenedor ejecutando `podman events`. Para ello en una terminal vamos a crear un contenedor rootless y en otra terminal ejecutamos `podman events`

En el primer terminal:

```
$ podman run ubuntu echo 'Hello world' 
Resolved "ubuntu" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/ubuntu:latest...
Getting image source signatures
Copying blob bccd10f490ab done   | 
Copying config ca2b0f2696 done   | 
Writing manifest to image destination
Hello world
```

Como es de esperar el usuario sin privilegio que crea el contenedor rootless no tiene guardada en su registro local la imagen `docker.io/library/ubuntu:latest` por lo que se la baja de nuevo.

Es importante tener en cuenta que cada usuario tiene su propio conjunto de imágenes descargadas, es decir:

* Si creamos contenedores rootful con el usuario `root`, las imágenes se guardarán en el directorio `/var/lib/containers/storage`.
* Si creamos contenedores rootless con el usuario `usuario`, las imágenes se guardarán en el directorio `/home/usuario/.local/share/containers/storage`.

En el segundo terminal (donde hemos ejecutado `podman events`) veremos las operaciones que se han dio produciendo:

```
2024-04-14 19:24:20.592277325 +0000 UTC image pull ca2b0f26964cf2e80ba3e084d5983dab293fdb87485dc6445f3f7bbfc89d7459 ubuntu
2024-04-14 19:24:20.956225765 +0000 UTC container create 80c2732ff6dbb93a9e95d0a218176a14bd9db537f200de92d3a192ba2b018785 (image=docker.io/library/ubuntu:latest, name=eloquent_noether, org.opencontainers.image.ref.name=ubuntu, org.opencontainers.image.version=22.04)
2024-04-14 19:24:21.502238172 +0000 UTC container init 80c2732ff6dbb93a9e95d0a218176a14bd9db537f200de92d3a192ba2b018785 (image=docker.io/library/ubuntu:latest, name=eloquent_noether, org.opencontainers.image.ref.name=ubuntu, org.opencontainers.image.version=22.04)
2024-04-14 19:24:21.544584186 +0000 UTC container start 80c2732ff6dbb93a9e95d0a218176a14bd9db537f200de92d3a192ba2b018785 (image=docker.io/library/ubuntu:latest, name=eloquent_noether, org.opencontainers.image.ref.name=ubuntu, org.opencontainers.image.version=22.04)
2024-04-14 19:24:21.602046252 +0000 UTC container attach 80c2732ff6dbb93a9e95d0a218176a14bd9db537f200de92d3a192ba2b018785 (image=docker.io/library/ubuntu:latest, name=eloquent_noether, org.opencontainers.image.ref.name=ubuntu, org.opencontainers.image.version=22.04)
2024-04-14 19:24:21.603927126 +0000 UTC container died 80c2732ff6dbb93a9e95d0a218176a14bd9db537f200de92d3a192ba2b018785 (image=docker.io/library/ubuntu:latest, name=eloquent_noether, org.opencontainers.image.ref.name=ubuntu, org.opencontainers.image.version=22.04)
2024-04-14 19:24:23.146601679 +0000 UTC container cleanup 80c2732ff6dbb93a9e95d0a218176a14bd9db537f200de92d3a192ba2b018785 (image=docker.io/library/ubuntu:latest, name=eloquent_noether, org.opencontainers.image.ref.name=ubuntu, org.opencontainers.image.version=22.04)
```