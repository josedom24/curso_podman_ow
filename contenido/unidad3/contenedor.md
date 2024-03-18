# Ejecución simple de contenedores

En este apartado vamos a crear un contenedor especificando el comando que debe ejecutar a partir de la imagen `ubuntu`.
En este caso vamos a descargar primero la imagen del registro público Docker Hub, y a continuación crearemos el contenedor.

Para descargar la imagen ejecutamos:

```bash
$ sudo podman pull ubuntu
Trying to pull docker.io/library/ubuntu:latest...
Getting image source signatures
Copying blob bccd10f490ab done   | 
Copying config ca2b0f2696 done   | 
Writing manifest to image destination
ca2b0f26964cf2e80ba3e084d5983dab293fdb87485dc6445f3f7bbfc89d7459
```

**Nota:** En este caso no indicamos el nombre del registro, ya que la imagen ubuntu esta definida en el fichero `/etc/containers/registries.conf.d/000-shortnames.conf` donde indicamos alias a los nombres de la imagen para que sea más fácil su gestión.

A continuación, creamos y ejecutamos un nuevo contenedor indicando el comando que va a ejecutar:

```bash
$ sudo podman run ubuntu echo 'Hello world'
Hello world
```

Comprobamos que el contenedor ha ejecutado el comando que hemos indicado y se ha parado:

```bash
$ sudo podman ps -a
CONTAINER ID  IMAGE                            COMMAND           CREATED        STATUS                    PORTS       NAMES
6e8e46cee350  docker.io/library/ubuntu:latest  echo Hello world  2 seconds ago  Exited (0) 2 seconds ago              eloquent_hodgkin
```

Los contenedores que hemos creado se nombran de manera aleatoria. Podemos cambiar el nombre de cualquier contenedor usando el comando `podman rename`:

```bash
$ sudo podman rename eloquent_hodgkin contenedor_ubuntu
```

Y comprobamos que el nombre ha cambiado:

```bash
$ sudo podman ps -a
CONTAINER ID  IMAGE                            COMMAND           CREATED             STATUS                         PORTS       NAMES
6e8e46cee350  docker.io/library/ubuntu:latest  echo Hello world  About a minute ago  Exited (0) About a minute ago              contenedor_ubuntu
```

## ¿Qué ocurre cuando creamos un contenedor?

Para terminar podemos ver las distintas etapas por las que pasa la creación de un contenedor ejecutando `podman events`. Para ello en una terminal ejecutamos el comando:

```bash
$ sudo podman events
```

Y en otro terminal ejecutamos un contenedor:

```bash
$ sudo podman run ubuntu echo 'Hello world' 
```

En el primer terminal veremos las operaciones que se han dio produciendo:

```
2024-03-18 08:27:46.280712472 +0000 UTC image pull ca2b0f26964cf2e80ba3e084d5983dab293fdb87485dc6445f3f7bbfc89d7459 ubuntu
2024-03-18 08:27:46.388517553 +0000 UTC container create d9873a66cbf1b0b81f9ffa96914cf2deedaa65cdacb94a27ad6c7ddd193e84ef (image=docker.io/library/ubuntu:latest, name=angry_austin, org.opencontainers.image.ref.name=ubuntu, org.opencontainers.image.version=22.04)
2024-03-18 08:27:46.986584664 +0000 UTC container init d9873a66cbf1b0b81f9ffa96914cf2deedaa65cdacb94a27ad6c7ddd193e84ef (image=docker.io/library/ubuntu:latest, name=angry_austin, org.opencontainers.image.ref.name=ubuntu, org.opencontainers.image.version=22.04)
2024-03-18 08:27:47.016922576 +0000 UTC container start d9873a66cbf1b0b81f9ffa96914cf2deedaa65cdacb94a27ad6c7ddd193e84ef (image=docker.io/library/ubuntu:latest, name=angry_austin, org.opencontainers.image.ref.name=ubuntu, org.opencontainers.image.version=22.04)
2024-03-18 08:27:47.054394528 +0000 UTC container attach d9873a66cbf1b0b81f9ffa96914cf2deedaa65cdacb94a27ad6c7ddd193e84ef (image=docker.io/library/ubuntu:latest, name=angry_austin, org.opencontainers.image.ref.name=ubuntu, org.opencontainers.image.version=22.04)
2024-03-18 08:27:47.055102924 +0000 UTC container died d9873a66cbf1b0b81f9ffa96914cf2deedaa65cdacb94a27ad6c7ddd193e84ef (image=docker.io/library/ubuntu:latest, name=angry_austin, org.opencontainers.image.ref.name=ubuntu, org.opencontainers.image.version=22.04)
2024-03-18 08:27:47.595430945 +0000 UTC container cleanup d9873a66cbf1b0b81f9ffa96914cf2deedaa65cdacb94a27ad6c7ddd193e84ef (image=docker.io/library/ubuntu:latest, name=angry_austin, org.opencontainers.image.ref.name=ubuntu, org.opencontainers.image.version=22.04)
```