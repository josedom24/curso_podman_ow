# Introducción a los contenedores rootless

Todos los ejemplos que hemos estudiado en los apartados anteriores han sido de contenedoires rootful. en este aprtado vamos a ejecutar nuestro primer contenedor rootless. Cómo hemos indicado anterioriormente, un contenedor rootless, es un contenedor que puede ejecutarse sin privilegios de `root` en el host. Por lo tanto los siguientes comandos lo vamos a ejecutar con un usuario sin privilegios, y por lo tanto no vamos a usar `sudo`.

## Ejecución de nuestro primer contenedor rootless

La manera de trabajar es similar a la anterior. En este caso podemos ejecutar con un usuario sin privilegios, el siguiente comando:

```
$ podman run hello-world
Resolved "hello-world" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull quay.io/podman/hello:latest...
...
!... Hello Podman World ...!
...
```

Como podemos observar para crear el contenedor se ha tenido que descargar la imagen. Es importante tener en cuenta que cada usuario tiene su propio conjunto de imágenes descargadas, es decir:

* Si creamos contenedores rootful con el usuario `root`, las imágenes se guardarán en el directorio `/var/lib/containers/storage`.
* Si creamos contenedores rootless con el usuario `usuario`, las imágenes se guardarán en el directorio `/home/usuario/.local/share/containers/storage`.

Vemos que sólo tenemos una imagen en nuestro registro local:

```
$ podman images
REPOSITORY                TAG         IMAGE ID      CREATED       SIZE
quay.io/podman/hello      latest      a4e07799a34b  37 hours ago  1.7 MB
```

Además, comprobamos que el contenedor se ha parado después de mostrar el mensaje de bienvenida.

```
$ podman ps -a
CONTAINER ID  IMAGE                            COMMAND               CREATED        STATUS                      PORTS       NAMES
1a729ea5ce72  quay.io/podman/hello:latest      /usr/local/bin/po...  4 minutes ago  Exited (0) 4 minutes ago                dreamy_fermat
```

Finalmente podemos eliminar el contenedor:

```
$ podman rm dreamy_fermat
dreamy_fermat
```

## Ejecución de contenedores rootless

De manera similar a la ejecución de contenedores roolful, podemos crear crear un contenedor rootless interactivo de la siguiente manera:

```
$ podman run -it --name contenedor1 alpine
...
/ #
```

Para crear un contenedor demonio, podemos ejecutar la siguiente instrucción:

```
podman run -d --name miweb -p 8080:80 quay.io/libpod/banner
```

* La imagen `quay.io/libpod/banner` es un servidor web con una página predeterminada. Es una imagen muy liviana.
* Al crear un contenedor rootless no podemos utilizar los puertos privilegiados (menores del 1024), por lo que hemos mapeado el puerto 8080.

Para comprobar que funciona, podemos acceder desde el mismo host. Por ejemplo, con la herramienta `curl`:

```
$ curl http://localhost:8080
   ___          __              
  / _ \___  ___/ /_ _  ___ ____ 
 / ___/ _ \/ _  /  ' \/ _ `/ _ \
/_/   \___/\_,_/_/_/_/\_,_/_//_/

```

En los aparatados posteriores iremos estudiando con detalle la ejecución de contenedores rootless, y profundizaremos sobre el almacenamiento y las redes en este tipo de contenedores.