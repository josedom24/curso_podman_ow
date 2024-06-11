# Construcción de imágenes desde cero con Buildah

Otra características que no ofrece Buildah es que puedes construir fácilmente una imagen de contenedor desde cero y por lo tanto excluir paquetes innecesarios de tu imagen. 

Para ello al crear el contenedor de trabajo vamos a usar una imagen especial que se llama `scratch` (realmente no es una imagen, es una forma de indicar que queremos empezar de cero en la creación del contenedor) y que le dice a Buildah que cree un contenedor con un sistema de archivo vacío. 

Posteriormente montaremos el sistema de archivo del contenedor y usando alguna herramienta específica de la distribución, copiaremos en él los archivos de un sistema base.

## Construcción de imagen con la distribución Fedora

En este ejemplo vamos a crear una nueva imagen utilizando un usuario sin privilegio (rootless). En primer lugar creamos un contenedor vacío usando la imagen `scratch`:

```
$ buildah --name contenedor-work2 from scratch

$ buildah ps
CONTAINER ID  BUILDER  IMAGE ID     IMAGE NAME                       CONTAINER NAME
43e74a80b8c5     *                  scratch                          contenedor-work2
```

Podemos comprobar que efectivamente el sistema de fichero del contenedor está vacío:

```
$ buildah run contenedor-work2 bash
2024-04-18T11:43:51.254896Z: executable file `bash` not found in $PATH: No such file or directory
error running container: from /usr/bin/crun creating container for [bash]: : exit status 1
...
```

A continuación vamos a montar la capa vacía que forma parte del contenedor usando `buildah mount`. En este caso, tendremos que tener en cuenta que necesitaremos usar `buildah unshare` si estamos montando el contenedor en un entorno rootless.

```
$ buildah unshare
# buildah mount contenedor-work2
/home/usuario/.local/share/containers/storage/overlay/6047f1b66e576cb7df3f51449db55d2c08413e8a2fce76d3ccc47e7b5d48bb93/merged
```

Obtenemos el directorio donde hemos montado el contenedor. A continuación usando una herramienta específica de la distribución que estemos usando copiaremos en ese directorio un sistema base. En Fedora, podríamos ejecutar la siguiente instrucción:

```
# dnf install --installroot /home/usuario/.local/share/containers/storage/overlay/6047f1b66e576cb7df3f51449db55d2c08413e8a2fce76d3ccc47e7b5d48bb93/merged \
  --releasever 39 bash coreutils \
  --setopt install_weak_deps=false -y
```

En este caso hemos instalado `bash` y `coreutils` de la distribución 39 de Fedora.

## Construcción de imagen con la distribución Debian/Ubuntu

En este ejemplo vamos a crear una nueva imagen utilizando un usuario con privilegio (rootful). En primer lugar creamos el contenedor de trabajo:

```
$ sudo su -
# buildah --name contenedor-work2 from scratch
# buildah mount contenedor-work2
/var/lib/containers/storage/overlay/0ada026fe64b76748c6c7ca0c78eeb73a695bbe088e67515f9e81e6ecaf8d4c7/merged
```

Si estamos trabajando con una distribución Debian o Ubuntu vamos a usar la herramienta `debootstrap`, que nos permite inicializar un sistema de archivos de Debian o Ubuntu. Para ello una vez montado el contenedor de trabajo, ejecutamos las siguientes instrucciones en otra terminal:

```
# apt install debootstrap 
# debootstrap bookworm /home/usuario/.local/share/containers/storage/overlay/6047f1b66e576cb7df3f51449db55d2c08413e8a2fce76d3ccc47e7b5d48bb93/merged
```

Al ejecutar `debootstrap` indicamos la distribución (`bookworm` si queremos Debian 12, `jammy` si queremos Ubuntu 22.04,...) y el directorio donde se copiaran los archivos, en nuestro caso el punto de montaje del contenedor.


## Configuración final para la construcción de la imagen desde cero

Una vez que hemos copiado el sistema de fichero base en el contenedor de trabajo, continuamos con el proceso. En este ejemplo continuamos con el contenedor de trabajo al que hemos instalado un sistema base de Fedora.

Ya podemos desmontar el contenedor, y salimos del modo `unsahare`:

```
# buildah unmount contenedor-work2
# exit
```

Podemos comprobar que efectivamente hemos instalado un sistema base en el contenedor:

```
$ buildah run contenedor-work2 bash
bash-5.2# 
```

A continuación vamos a terminar de configurar el contenedor, creando un pequeño script que va mostrar un mensaje:

```
$ buildah run contenedor-work2 bash -c 'echo "echo \"Contenedor desde scratch!!!\"" > /usr/bin/runecho.sh'
$ buildah run contenedor-work2 chmod +x /usr/bin/runecho.sh
$ buildah config --cmd "sh /usr/bin/runecho.sh" contenedor-work2
```
Y creamos una nueva imagen:

```
$ buildah commit contenedor-work2 josedom24/fedora-base:latest
```

Y comprobamos le ejecución de un contenedor a partir de la imagen que hemos creado:

```
$ podman run --rm josedom24/fedora-base
Contenedor desde scratch!!!
```
