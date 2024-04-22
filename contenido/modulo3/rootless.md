# Almacenamiento de contenedores rootless

Aunque en los apartados anteriores hemos estudiado cómo se almacenan las imágenes y contenedores de forma general, en este apartado veremos algunos aspectos específicos en el almacenamiento de imágenes y contenedores cuando trabajamos con usuarios sin privilegios.

Para obtener el aislamiento del sistema de ficheros que evita conflictos vamos a utiliza un espacio de nombre de montaje (**mount namespace**) que aísla los puntos del montaje del contenedor con los del host.

Además con el uso de los espacios de nombre de montaje en combinación con el espacio de nombres del usuario vamos a conseguir las siguientes características:

* Nos va a permitir **aislar los puntos de montaje del contenedor con los del host**. Cuando se ejecuta Podman en modo rootless, cada contenedor se ejecuta con su propio espacio de nombres de montaje. Esto significa que los puntos de montaje dentro del contenedor se aíslan del sistema de archivos del host, proporcionando un nivel de aislamiento adicional y seguridad.
* Nos permite a **usuarios sin privilegios realizar puntos de montaje**. Los usuarios sin privilegios pueden realizar puntos de montaje dentro de sus propios espacios de nombres de montaje. Esto les permite montar sistemas de archivos dentro de los contenedores sin necesidad de privilegios de `root`.
* Nos permite **guardar las capas de las imágenes con los usuarios y grupos apropiados**. Se manejan de forma adecuada los permisos de archivos y directorios dentro de las capas de imágenes para garantizar que se guarden con los usuarios y grupos apropiados. 

Para comprobarlo, hemos creado dos contenedores rootless:

```
$ podman run -d --rm --name contenedor1 alpine sleep 1000
$ podman run -d --rm --name contenedor2 alpine sleep 1000

$ podman ps
CONTAINER ID  IMAGE                            COMMAND     CREATED        STATUS        PORTS       NAMES
57a4a26bc9c2  docker.io/library/alpine:latest  sleep 1000  6 seconds ago  Up 6 seconds              contenedor1
41546a56eb63  docker.io/library/alpine:latest  sleep 1000  3 seconds ago  Up 3 seconds              contenedor2
```

Los puntos de montaje de estos contenedores están aislados de los del host:
```
$ mount | grep overlay
```

Como vimos, la instrucción `podman unshare`, nos permite ejecutar instrucciones entrando en el espacio de nombres de usuario. Además, también entramos en el espacio de nombres de montaje, por lo tanto podemos ver los dos montajes de los dos contenedores:

```
$ podman unshare mount | grep overlay
...
overlay on /home/usuario/.local/share/containers/storage/overlay/203cac195f25541ff30f4abc9b79ad972e6478f233499be6caf4a6a405d2cea9/merged ...

overlay on /home/usuario/.local/share/containers/storage/overlay/b2ff6deab57f9e87c7cbd25cc11ce1ec05be11aab09e14faf4e0648693ef0bb7/merged ...
```

Evidentemente dentro de cada contenedor sólo es visible su punto de montaje:

```
$ podman exec contenedor1 mount|grep overlay
overlay on / type overlay (rw,context="system_u:object_r:container_file_t:s0:c118,c776",relatime,...
```

Además podemos ver los ficheros que forman parte del sistema de fichero con los usuarios definidos dentro del contenedor. Para ello primero vamos a obtener el directorio donde se ha unido el sistema de archivos de unión:

```
$ podman inspect --format='{{range $key,$dir := .GraphDriver.Data}}{{$key}} = {{$dir}}\n{{end}}'  contenedor1
...
MergedDir = /home/usuario/.local/share/containers/storage/overlay/203cac195f25541ff30f4abc9b79ad972e6478f233499be6caf4a6a405d2cea9/merged
...
```

Y ahora podemos listar los ficheros de ese directorio, utilizando `podman unshare` para acceder al espacio de nombres de usuario y el espacio de nombres de montaje:

```
$ podman unshare ls -l /home/usuario/.local/share/containers/storage/overlay/203cac195f25541ff30f4abc9b79ad972e6478f233499be6caf4a6a405d2cea9/merged
total 0
drwxr-xr-x. 1 root root 858 Jan 26 17:53 bin
drwxr-xr-x. 1 root root   0 Jan 26 17:53 dev
...
```

