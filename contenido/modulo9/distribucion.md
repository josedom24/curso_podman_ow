# Distribución de imágenes con Buildah

Con Buildah también podemos distribuir las imágenes que hemos creado. Por lo tanto tenemos la misma funcionalidad que usando Podman:

## Logueo en los registros

Si queremos subir una imagen a un registro remoto, tendremos que loguearnos en el registro, por ejemplo:

```
$ buildah login docker.io
$ buildah login quay.io
```

Si queremos saber si estamos logueado en algún registro:

```
$ buildah login --get-login docker.io
$ buildah login --get-login quiay.io
```

## buildah push

`buildah push` lo utilizaremos normalmente para subir una imagen de nuestro registro local a un registro remoto. Pero en realidad, `buildah push` puede guardar una imagen usando cualquier transporte de imagen. Veamos algunos ejemplos:

* Para subir una imagen que hemos construido a un registro remoto. Recuerda que el nombre de la imagen tiene que tener como primera parte el nombre del usuario con el que nos hemos logueado al registro:

    ```
    $ buildah push localhost/josedom24/debian-apache:latest quay.io/josedom24/debian-apache:latest
    ```
    Se está utilizando el transporte de imágenes por defecto, que es `docker` que hace referencia a una imagen almacenada en un registro remoto de imágenes.

* Además de almacenar las imágenes en un registro remoto, `buildah push` puede guardar una imagen local en un fichero o en un directorio:
    ```
    $ buildah push localhost/josedom24/debian-apache:latest oci-archive:debian-apache.tar
    $ buildah push localhost/josedom24/debian-apache:latest oci:debian-apache
    ```
 
## buildah pull

`buildah pull` lo utilizaremos normalmente para descargar una imagen de un registro remoto a nuestro registro local:

```
$ buildah pull quay.io/josedom24/debian-apache:latest
```

Pero usando los distintos transportes de imágenes también podemos guardar una imagen a nuestro registro local a partir de un fichero o un directorio:

```
$ podman pull oci-archive:debian-apache.tar
$ podman pull oci:debian-apache
```

En este caso no se nombra la imagen de forma correcta y la tenemos que renombrar a partir de su ID:

```
$ buildah tag a93fcf5cd27e quay.io/josedom24/debian-apache
