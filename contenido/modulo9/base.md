# Construcción de imágenes desde una imagen base con Buildah

Buildah utiliza el concepto de **Contenedor de Trabajo**. 

Vamos a crear un contenedor a partir de una imagen base. Sobre este contenedor le podemos realizar distintas operaciones: ejecución de comandos, copiar ficheros, configurar distintos parámetros,...

Finalmente a partir de este contenedor de trabajo crearemos nuestra nueva imagen.

## Construcción de imágenes desde una image base

Para crea un contenedor de trabajo podemos ejecutar el siguiente comando, indicando la imagen base que vamos a usar:

```
$ buildah from --name contenedor-work1 debian:12
```

Podemos ver los contenedores de trabajo que tenemos creados, ejecutando:

```
$ buildah ps
CONTAINER ID  BUILDER  IMAGE ID     IMAGE NAME                       CONTAINER NAME
5730a5e9d8be     *     e15dbfac2d2b docker.io/library/debian:12      contenedor-work1
```

Ahora podemos ejecutar comandos en el contenedor de trabajo, usando `buildah run`, por ejemplo:

```
$ buildah run contenedor-work1 bash -c "apt update && apt install -y apache2"
```

Si tenemos ficheros en nuestro host podemos copiarlos al contenedor de trabajo usando `buildah copy`:

```
$ echo "<h1>Podman</h1>">index.html
$ buildah copy contenedor-work1 index.html /var/www/html/index.html
```

También podemos configurar los metadatos del contenedor, en este caso vamos a indicar el comando que se ejecutará por defecto y el puerto que se utiliza el servicio. Para ello usamos `buildah config`:

```
$ buildah config --cmd "/usr/sbin/apache2ctl -DFOREGROUND" contenedor-work1
$ buildah config -p 80 contenedor-work1
```

Finalmente, usando `buildah commit` podemos crear nuestra nueva imagen:

```
$ buildah commit contenedor-work1 josedom24/debian-apache:latest
```

Comprobamos que la imagen se ha creado:

```
$ buildah images
REPOSITORY                         TAG          IMAGE ID      CREATED        SIZE
localhost/josedom24/debian-apache  latest       be49bd721e9f  3 seconds ago  261 MB
```

También podemos inspeccionar una imagen:

```
$ buildah inspect josedom24/debian-apache:latest
```

El contenedor de trabajo continúa ejecutándose, cuando terminemos de trabajar con él lo podemos eliminar ejecutando:

```
$ buildah rm contenedor-work1
```

Finalmente si queremos eliminar una imagen, ejecutamos:

```
$ buildah rmi josedom24/debian-apache:latest
```




