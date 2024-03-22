# Almacenamiento de imágenes

Las imágenes de contenedor se construyen a partir de de dos elementos:

* Una configuración: donde se guarda la metainformación de la imagen, por ejemplo se indica el comando que se ejecuta en el contenedor que ejecutemos a a partir de la imagen.
* Un sistema de archivos de unión, que estará formado por varias **capas ordenadas**. 

Veamos las implicaciones que tiene el uso de sistemas de archivos de unión en la estructura de una imagen de contenedor:

* Como hemos estudiado anteriormente, podemos pensar en una capa como un conjunto de cambios en el sistema de archivos. Es decir, un conjunto de diferencias con respecto a la capa anterior que se guardan en diferentes directorios.
* En el proceso de creación de las imágenes, los comandos que cambian el sistema de archivos (instalaciones, modificación de ficheros, copiar ficheros,...) producen una nueva capa.
* Si tienes muchas imágenes basadas en capas similares (capas que contienen sistemas operativos similares o ficheros comunes), entonces todas estas capas comunes serán almacenadas sólo una vez.

## Especificación de imagen OCI

Open Container Initiative (OCI) es la organización responsable de estandarizar las especificaciones referentes al trabajo con los contenedores. Una de las especificaciones que desarrolla es el **formato de imagen** (Open Container Initiative Image Format normalmente abreviado en OCI Image Format). Determina el formato para empaquetar la imagen del contenedor de software. Con esto conseguimos que distintas aplicaciones (motores de contenedores, registros de imágenes,...) puedan trabajar con el mismo formato de imágenes.

El formato de las imágenes Docker, llamado **Docker V2** difiere al formato de imágenes OCI, pero son totalmente compatibles.

Otra de las especificaciones desarrolladas por esta entidad es cómo se distribuyen las imágenes y como se almacenan en los registros de imágenes. La especificación OCI de distribución de imágenes está basada en la especificación Docker de distribución y es la que vamos a presentar en este apartado.

En resumen, estas especificaciones estandarizan la manera en que se construyen y almacenan las imágenes de contenedores: dónde se guarda su configuración y cómo se almacena su sistema de ficheros en distintas capas para posteriormente crear el sistema de archivos del contenedor usando un driver de almacenamiento.

## Almacenamiento de una imagen OCI en Podman

Podman puede usar varios drivers para gestionar el sistema de archivos de unión que constituye una imagen: overlay, vfs, devmapper, aufs, btrfs, zfs,...

El driver de almacenamiento que se usa por defecto para gestionar el conjunto de capas que forman parte de una imagen en la versión actual de Podman es **Overlay v2**. 

El fichero de configuración de Podman donde se indica la configuración del almacenamiento es:

* Para contenedores rootful: `/usr/share/containers/storage.conf`.
* Para contenedores rootless creados por el usuario `usuario`: `/home/usuario/.config/containers/storage.conf`

Los parámetros más importantes de estos ficheros son `driver`, donde se indica el driver utilizado y `graphroot` donde se indica el directorio de almacenamiento de las imágenes. Este directorio será el siguiente:

* Para contenedores rootful: `/var/lib/containers/storage`.
* Para contenedores rootless creados por el usuario `usuario`: `/home/usuario.local/share/containers/storage/`.

La información de la configuración de almacenamiento, la podemos ver ejecutando el siguiente comando para contenedores rootful:

```bash
$ sudo podman info|grep -A19 store
store:
  configFile: /usr/share/containers/storage.conf
  ...
  graphDriverName: overlay
  ...
  graphRoot: /var/lib/containers/storage
  ...
```

Y para contenedores rooless:

```bash
$ podman info|grep -A19 store
store:
  configFile: /home/usuario/.config/containers/storage.conf
  ...
  graphDriverName: overlay
  ...
  graphRoot: /home/usuario.local/share/containers/storage/
  ...
```

## Ejemplo de almacenamiento imagen

Para ver este ejemplo, vamos a descargar una imagen para crear un contenedor rootful y veremos la estructura de los directorios donde se almacena. En primer lugar descargamos una imagen:

```bash
$ sudo podman pull quay.io/centos7/httpd-24-centos7:latest
Trying to pull quay.io/centos7/httpd-24-centos7:latest...
Getting image source signatures
Copying blob 8f001c8d7e00 done   | 
Copying blob c61d16cfe03e done   | 
Copying blob 06c7e4737942 done   | 
Copying config d7af31210b done   | 
Writing manifest to image destination
d7af31210b288164c319bae740ca1281528390a3c5cee657e95f243670b49e6a
```

Como observamos esta imagen está formado por 3 capas (`Copying blob...`) y su configuración (`Copying config...`).

Veamos la estructura de directorio que tenemos en el directorio de almacenamiento, como `root` ejecutamos las siguientes instrucciones:

```bash
# cd /var/lib/containers/storage/
# ls
db.sql  defaultNetworkBackend  libpod  overlay  overlay-containers  overlay-images  overlay-layers  secrets  storage.lock  tmp  userns.lock  volumes
```
Los directorios que nos interesan son los siguientes:

* `overlay-images`: Contiene la configuración (los metadatos) de las imágenes descargadas.
* `overlay-layers`: Contiene los archivos comprimidos de todas las capas de las imágenes que tenemos descargadas.
* `overlay`: Este es el directorio que contiene las capas descomprimidas de cada imagen.

### Directorio overlay-images

Veamos el directorio `overlay-images`:

```
# cd overlay-images/
# ls
d7af31210b288164c319bae740ca1281528390a3c5cee657e95f243670b49e6a  images.json  images.lock
```

Los ficheros y directorios que nos encontramos son los siguientes:

* `images.json`: Es un índice de las imágenes que tenemos descargas. Para visualizarlo de manera correcta este fichero podemos usar la utilidad `jq`: `cat images.json | jq`.
* Distintos directorios que corresponde a las imágenes que tenemos descargadas en nuestro registro local. El nombre de estos directorios corresponden a los identificados de las imágenes. En estos directorios encontramos:
  * Un fichero de manifiesto llamado `manifest`, que describe las capas que componen la imagen.
  * Y distintos ficheros con las configuración de la imagen.

Podemos ver las capas que forman parte de la imagen que hemos descargado, ejecutando:

```bash
$ sudo cat overlay-images/d7af31210b288164c319bae740ca1281528390a3c5cee657e95f243670b49e6a/manifest | jq
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
  "config": {
    "mediaType": "application/vnd.docker.container.image.v1+json",
    "size": 15278,
    "digest": "sha256:d7af31210b288164c319bae740ca1281528390a3c5cee657e95f243670b49e6a"
  },
  "layers": [
    {
      "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
      "size": 78951426,
      "digest": "sha256:c61d16cfe03e7bfb4e7e312f09fb17a815be72096544133320058ee6ce55d0b2"
    },
    {
      "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
      "size": 10435798,
      "digest": "sha256:06c7e47379429b2a921140524d1596e2c2bf8bc7b29fa9df0ee73c91f5b4c24f"
    },
    {
      "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
      "size": 50603928,
      "digest": "sha256:8f001c8d7e009adf9e088ff8b85806da558aa713eb545d2af045943eed1ad66a"
    }
  ]
}
```

Como vemos la imagen está formada por una configuración y por un conjunto de capas ordenadas, que están referenciada con un hash.

Podemos verlo de manera gráfica:

![images1](img/images1.png)


### Directorio overlay-layers

Veamos el directorio `overlay-layers`:

```
# cd overlay-layers/
# ls
007d2037805f6ca87f969f06c81286a47d98664e3f62e5fd393ec3da08a55b3c.tar-split.gz  layers.json
53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a4483b51e232309aee.tar-split.gz  layers.lock
8853b21ed9ab4ab7fd6c118f5b1c11e974caa7e133a99981573434d3b6018cf0.tar-split.gz
```

Como podemos ver, acabamos de encontrar todos los archivos de capas descargados de nuestra imagen. Además el fichero `layers.json` es un índice de todas las capas que tenemos descargadas:

```bash
$ sudo cat overlay-layers/layers.json | jq
[
  {
    "id": "53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a4483b51e232309aee",
    "created": "2024-03-21T07:38:50.395783286Z",
    "compressed-diff-digest": "sha256:c61d16cfe03e7bfb4e7e312f09fb17a815be72096544133320058ee6ce55d0b2",
    "compressed-size": 78951426,
    "diff-digest": "sha256:53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a4483b51e232309aee",
    "diff-size": 211829760,
    "compression": 2,
    ...
```

Además del hash que vimos que identifica a la capa, encontramos también el campo `id` que es una cadena única que también nos permite referenciar a la capa.

De manera gráfica:

![images2](img/images2.png)

### Directorio overlay

Las capas que hemos visto anteriormente están descomprimidas en el directorio `overlay`:

```
# cd overlay
# ls
007d2037805f6ca87f969f06c81286a47d98664e3f62e5fd393ec3da08a55b3c  8853b21ed9ab4ab7fd6c118f5b1c11e974caa7e133a99981573434d3b6018cf0
53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a4483b51e232309aee  l
```

En primer lugar tenemos un directorio por cada una de las capas descargadas, estos directorios serán los que unamos para crear un sistema de archivos de unión al crear un contenedor. La estructura de estos directorios depende del orden de la capa, si vemos el contenido de una capa que no es la primera, tenemos la siguiente estructura:

```
# cd 007d2037805f6ca87f969f06c81286a47d98664e3f62e5fd393ec3da08a55b3c/
# ls
diff  link  lower  merged  work
```

Veamos que guardan cada uno de estos directorios y ficheros:

* `diff`: Este directorio representa la capa superior de la superposición, y se utiliza para almacenar
los cambios en la capa.
* `lower`: 
* `merged`: En este directorio se monta el sistema de fichero superpuesto.
* `work`: Este directorio se utiliza para operaciones internas durante el montaje.
* `link`: Este archivo contiene un identificador de capa abreviados.

En el directorio `overlay` también encontramos un directorio `l`. En este directorio hay enlaces simbólicos con identificadores de capa abreviados que apuntan al directorio `diff` para cada capa. Los enlaces simbólicos hacen referencia a capas inferiores (los identificadores de capa abreviados indicadas en el fichero `link` de cada capa).

Se utilizan identificadores de capa abreviados y no los propios hashes de las capas para que el montaje sea máq eficiente, ya que los hashes son cadenas más grandes que los identificados abreviados y por lo tanto su gestión es más eficiente a la hora de realizar el montaje.

```
# cd overlay/l
# ls -al
total 12
drwxr-xr-x. 1 root root 156 Mar 21 07:45 .
drwx------. 1 root root 422 Mar 21 07:45 ..
lrwxrwxrwx. 1 root root  72 Mar 21 07:38 IVBKXQVXCMS3S4MYZYTY4NQ3W5 -> ../007d2037805f6ca87f969f06c81286a47d98664e3f62e5fd393ec3da08a55b3c/diff
lrwxrwxrwx. 1 root root  72 Mar 21 07:38 LCIWXBIPSMIGB2RTQV36QKTCRH -> ../53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a4483b51e232309aee/diff
lrwxrwxrwx. 1 root root  72 Mar 21 07:39 ZXSJGMR5T7VDVVGRWHG3E2I6DZ -> ../8853b21ed9ab4ab7fd6c118f5b1c11e974caa7e133a99981573434d3b6018cf0/diff
```

Volvamos a ver el manifiesto de la imagen con la que estamos trabajando:

```
# cd overlay-images/d7af31210b288164c319bae740ca1281528390a3c5cee657e95f243670b49e6a/
# cat manifest | jq
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
  "config": {
    "mediaType": "application/vnd.docker.container.image.v1+json",
    "size": 15278,
    "digest": "sha256:d7af31210b288164c319bae740ca1281528390a3c5cee657e95f243670b49e6a"
  },
  "layers": [
    {
      "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
      "size": 78951426,
      "digest": "sha256:c61d16cfe03e7bfb4e7e312f09fb17a815be72096544133320058ee6ce55d0b2"
    },
    {
      "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
      "size": 10435798,
      "digest": "sha256:06c7e47379429b2a921140524d1596e2c2bf8bc7b29fa9df0ee73c91f5b4c24f"
    },
    {
      "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
      "size": 50603928,
      "digest": "sha256:8f001c8d7e009adf9e088ff8b85806da558aa713eb545d2af045943eed1ad66a"
    }
  ]
}
```

Encontramos que nuestra imagen está formada por 3 capas, de cada capa tenemos un hash que la identifica la capa descargada comprimida. 

A continuación, debemos comparar el hash de cada capa con la lista de todas las capas que hemos descargado:

```
# cd overlay-layers/
# cat layers.json | jq
[
  {
    "id": "53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a4483b51e232309aee",
    "created": "2024-03-21T07:38:50.395783286Z",
    "compressed-diff-digest": "sha256:c61d16cfe03e7bfb4e7e312f09fb17a815be72096544133320058ee6ce55d0b2",
    "compressed-size": 78951426,
    "diff-digest": "sha256:53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a4483b51e232309aee",
    "diff-size": 211829760,
    "compression": 2,
    ...
```

Haciendo la comprobación de hash encontramos el identificados de nuestra primera capa, y podemos acceder a su contenido:

```
# cd overlay/53498d66ad83a29fcd7c7bcf4abbcc0def4fc912772aa8a4483b51e232309aee
# ls
diff  empty  link  merged  work
```

Como podemos comprobar, no hay ningún archivo `lower` dentro del directorio de la capa porque esta es la
primera capa de nuestra imagen. La diferencia que podemos notar es la presencia de un directorio llamado `empty`. Esto se debe a que si una capa no tiene padre, el sistema de superposición creará un directorio vacío.
