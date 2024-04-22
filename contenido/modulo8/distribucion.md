# Distribución de imágenes OCI

Podman nos ofrece varias instrucciones para usar los **transportes de imágenes OCI** para distribuir una imagen OCI:

## Distribución de imágenes usando ficheros o directorios

### podman save

* Podemos guardar una imagen en un fichero o en un directorio usando la instrucción `podman save`. 
* `podman save` trabaja con distintos transportes:
    * **docker-archive**: Imagen con formato Docker comprimida en una archivo tar. Es el transporte por defecto, y es el que debemos usar si posteriormente queremos recuperar la imagen con `docker load`.
    * **oci-archive**: Imagen con formato OCI comprimida en una archivo tar.
    * **docker-dir**: Imagen con formato Docker, guardada en un directorio local.
    * **oci-dir**: Imagen con formato OCI, guardada en un directorio local.

Por ejemplo, podemos guardar una imagen `alpine` que tenemos en nuestro registro local:

```
$ podman save docker.io/alpine:latest > alpine.tar
```
En este caso el fichero `alpine.tar` guarda una imagen en formato Docker. Si queremos guardar una imagen con formato OCI:

```
$ podman save docker.io/alpine:latest --format oci-archive > alpine2.tar
```

Además `podman save` puede guardar una imagen en un directorio, con formato Docker, usando el formato `docker-dir`, o con formato OCI, usando el formato `oci-dir`.
    
```
$ podman save docker.io/alpine:latest --format docker-dir -o alpine-docker
$ podman save docker.io/alpine:latest --format oci-dir -o alpine-oci
```

### podman load

Utilizamos `podman load` para cargar una nueva imagen en nuestro registro local a partir de un fichero comprimido con la imagen.

```
$ podman rmi docker.io/alpine
$ podman load -i alpine.tar
```

Hemos recuperado la imagen con formato Docker, pero también podríamos haber recuperado la imagen con formato OCI.

## Distribución de imágenes usando registros

Como hemos indicado podemos trabajar con varios registros de imágenes. Si los registros son públicos hemos usado `podman pull` para descargar imágenes. Si queremos subir nuestras imágenes usaremos `podman push` pero necesitaremos registrarnos en los registros y tener un usuario que nos permita subir nuestras imágenes.

### Logueo en los registros

Si queremos subir una imagen a un registro remoto, tendremos que loguearnos en el registro, por ejemplo:

```
$ podman login docker.io
$ podman login quay.io
```

Si queremos saber si estamos logueado en algún registro:

```
$ podman login --get-login docker.io
$ podman login --get-login quiay.io
```

### podman push

`podman push` lo utilizaremos normalmente para subir una imagen de nuestro registro local a un registro remoto. Pero en realidad, `podman push` puede guardar una imagen usando cualquier transporte de imagen. Veamos algunos ejemplos:

* Para subir una imagen que hemos construido a un registro remoto. Recuerda que el nombre de la imagen tiene que tener como primera parte el nombre del usuario con el que nos hemos logueado al registro:

    ```
    $ podman push localhost/josedom24/webserver:v1 quay.io/josedom24/webserver:v1
    ```
    Se está utilizando el transporte de imágenes por defecto, que es `docker` que hace referencia a una imagen almacenada en un registro remoto de imágenes.

    Todas las imágenes subidas a `docker.io` son públicas y se podrán descargar con `podman pull`, sin embargo si subimos una imagen a `quay.io` por defecto son privadas y la tendremos que hacer públicas en **Settings->Make Public**.

* Además de almacenar las imágenes en un registro remoto, `podman push` puede guardar una imagen local en un fichero o en un directorio, del mismo modo que `podman save`:
    ```
    $ podman push docker.io/alpine oci-archive:alpine3.tar
    $ podman push docker.io/alpine oci:alpine
    ```
 
### podman pull

`podman pull` lo utilizaremos normalmente para descargar una imagen de un registro remoto a nuestro registro local:

```
$ podman pull quay.io/josedom24/webserver:v1
```

Pero usando los distintos transportes de imágenes también podemos guardar una imagen a nuestro registro local a partir de un fichero o un directorio, de manera similar a `podman load`:

```
$ podman pull oci-archive:alpine3.tar
$ podman pull oci:alpine
```

En este caso no se nombra la imagen de forma correcta y la tenemos que renombrar a a partir de su ID:

```
$ podman image tag bc4e4f799925 docker.io/alpine:latest
```