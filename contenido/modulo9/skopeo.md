# Gestión de imágenes con Skopeo

Skopeo nos permite realizar distintas operaciones sobre imágenes almacenadas en registros. Si trabajamos con imágenes almacenadas en registros remotos no es necesario que se la descargue al registro local. 

## Inspeccionar imágenes

Skopeo es capaz de inspeccionar un repositorio en un registro de imágenes y obtener información de las versiones de las imágenes que contiene. Por ejemplo, con el comando `list-tags` vemos la lista de etiquetas que tiene una imagen:

```
$ skopeo list-tags docker://quay.io/josedom24/servidorweb
{
    "Repository": "quay.io/josedom24/servidorweb",
    "Tags": [
        "v1",
        "v2",
        "latest"
    ]
}
```

Hemos indicado la imagen indicando el transporte de imagen, es este caso `docker` que hace referencia a un registro remoro de imágenes.

Ahora podemos inspeccionar la imagen, ejecutando:

```
$ skopeo inspect docker://quay.io/josedom24/servidorweb:latest
```

También podemos obtener la configuración del contenedor que podemos crear a partir de esta imagen:

```
$ skopeo inspect --config docker://quay.io/josedom24/servidorweb:latest
```


También podemos inspeccionar una imagen que estará guardada usando otro tranasporte de imagen, por ejemplo para inspeccionar una imagen guarda en un fichero, podríamos ejecutar:

```
$ skopeo inspect oci-archive:debian-apache.tar
```

## Copiar imágenes

Skopeo puede copiar imágenes utilizando los distintos transportes de imágenes. Si vamos a copiar una imagen a un regiistro remoto, tendremos que loguearnos en el registro. Vemos algunos ejemplos:

* Copiar una imagen desde un registro remoto a otro registro remoto:

    ```
    $ skopeo login quay.io
    $ skopeo copy docker://docker.io/josedom24/2048:v1 docker://quay.io/josedom24/2048:v1
    ```

    Recuerda que la imagen no se descarga al registro local. Podemos obtener información de la imagen que hemos copiado:

    ```
    $ skopeo inspect docker://quay.io/josedom24/2048:v1
    ```

* Copiar una imagen de un registro remoto al registro local:

    ```
    $ skopeo copy docker://quay.io/josedom24/hola-mundo:latest containers-storage:quay.io/josedom24/hola-mundo:latest
    ```

    Hemos utilizado el transporte de imagen `container-storage` que hace referencia a una imagen guardada en un registro local de Podman. Sería similar a `podman pull quay.io/josedom24/hola-mundo:latest`

* Copiar una imagen desde nuestro registro local a un registro remoto:

    ```
    skopeo copy containers-storage:quay.io/josedom24/hola-mundo:latest docker://docker.io/josedom24/hola-mundo:latest
    ```

    Sería similar a `podman push quay.io/josedom24/hola-mundo:latest`.

* También podemos copiar una imagen a un fichero o un directorio con un formato determinado:

    ```
    $ skopeo copy docker://quay.io/josedom24/hola-mundo:latest oci-archive:hola-mundo.tar
    $ skopeo copy containers-storage:quay.io/josedom24/hola-mundo:latest oci-archive:hola-mundo
    ```

    En el primer ejemplo hemos copiado una imagen desde un registro remoto (transporte `docker`) a un fichero comprimido con formato OCI (transporte `oci-archive`).
    En el segundo ejemplo hemos copiado una imagen desde el registro local (transporte `containers-storage`) a un directorio con formato OCI  (transporte `oci`).


## Eliminar imágenes

Desde Skopeo podemos eliminar imágenes. Podemos eliminar imágenes guardadas en el registro local:

```
$ skopeo delete containers-storage:quay.io/josedom24/hola-mundo:latest
```

Y podemos eliminar imágenes guardas en registros remotos:

```
$ skopeo delete docker://quay.io/josedom24/2048:v1
```

En este caso se elimina la imagen con la etiqueta indicada, pero el repositorio no se elimina. Tenemos que acceder a `quay.io` para eliminar el repositorio manualmente.

## Sincronizar imágenes

Podemos sincronizar todas las versiones de una imagen indicando el transporte de imágenes de origen y de destino. 

```
$ skopeo sync --src docker --dest docker docker.io/josedom24/2048 quay.io/josedom24
```
Indicamos el transporte de origen en el parámetro `--src`, el transporte de destino en el parémetro de destino `--dest`. En este caso se han copiado doos imágenes con dos etiquetas:
```
$ skopeo list-tags docker://quay.io/josedom24/2048
{
    "Repository": "quay.io/josedom24/2048",
    "Tags": [
        "v1",
        "v2"
    ]
}
```

