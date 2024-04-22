# Gestión de imágenes

Para crear un contenedor es necesario usar una imagen que tengamos descargada en nuestro registro local. Por lo tanto al ejecutar `podman run` se comprueba si tenemos la versión indicada de la imagen y si no es así, se procede a su descarga desde un **Registro remoto de imágenes**.

Podman puede trabajar con distintos registros remotos de imágenes que cumplan las especificaciones OCI de almacenamiento y distribución de imágenes.

## Descarga de imágenes

Otra manera de descargar una imagen a nuestro registro local, es usando la instrucción `podman image pull` o `podman pull` e indicando el nombre completo de la imagen qué como hemos visto estará formada por: el nombre del registro, el nombre de la imagen y opcionalmente, el nombre de la etiqueta. Por ejemplo: 

```
$  podman pull docker.io/nginx:stable
```

Recordamos que las imágenes descargadas con el usuario `root` se guardarán, por defecto, en el directorio `/var/lib/containers/storage`, si se descargan con un usuario sin privilegios , se guardarán en `$HOME/.local/share/containers/storage/`.

## Listado de imágenes locales

Para mostrar las imágenes que tenemos en nuestro registro local podemos usar `podman image ls` o la siguiente instrucción:

```
$ podman images
```

## Uso de nombre cortos de imágenes

Si al referenciar a una imagen usamos su nombre corto, es decir no indicamos el nombre del registro, indicando sólo el nombre de la imagen y la etiqueta, Podman no podrá determinar de que registro lo tiene que descargar. En este caso puede ocurrir dos cosas:

1. Que el nombre este declarado como un alias en el fichero `/etc/containers/registries.conf.d/000-shortnames.conf` y lo podrá descargar sin ningún problema. Por ejemplo:

    ```
    $ podman pull debian
    Resolved "debian" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
    Trying to pull docker.io/library/debian:latest...
    Getting image source signatures
    Copying blob 71215d55680c done   | 
    Copying config c978d997d5 done   | 
    Writing manifest to image destination
    c978d997d5fe02d50ae8b7c1e4338f3fcdb61bcf9940a8ce8f87811a319ce586
    ```
2. Que no haya definido un alias. En esta caso, Podman nos ofrecerá la posibilidad de elegir descargarla de alguno de los registros que tiene configurado en el parámetro `unqualified-search-registries` del fichero `/etc/containers/registries.conf`:

    ```
    unqualified-search-registries = ["registry.fedoraproject.org", "registry.access.redhat.com", "docker.io", "quay.io"]
    ```

    Por ejemplo:

    ```
    # podman pull nginx
    ? Please select an image: 
      ▸ registry.fedoraproject.org/nginx:latest
        registry.access.redhat.com/nginx:latest
        docker.io/library/nginx:latest
        quay.io/nginx:latest
    ```

    Al elegir la imagen concreta que queremos descargar, se creará un alias en el fichero `/var/cache/containers/short-name-aliases.conf` si estamos trabajando con el usuario `root`o en el fichero `$HOME/.cache/containers/short-name-aliases.conf` si trabajamos con un usuario sin privilegios.

## Borrar una imagen

Si queremos borrar una imagen, usaremos la instrucción `podman image rm` o la siguiente instrucción:

```
$ podman rmi docker.io/nginx
```
No podemos eliminar una imagen si tenemos algún contenedor creado a partir de ella.

Para eliminar todas las imágenes sin contenedores asociados:

```
$ podman rmi --all
```

