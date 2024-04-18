# Construcción de imágenes desde un Containerfile con Buildah

Buildah nos permite también crear imágenes a partir de fichero `Containerfile` o `Dockerfile` y un contexto de manera similar a como lo hacemos con `podman build`. Puedes encontrar los ficheros necesarios en el [Repositorio con el código de los ejemplos](xxx).

En este ejemplo vamos a crear una imagen a partir del siguiente `Containerfile`:

```
FROM registry.access.redhat.com/ubi8/python-38:latest
WORKDIR /app
COPY app .
RUN python -m pip install -r requirements.txt
USER 1001 
EXPOSE 5000
CMD ["python3", "app.py"]
```

* La imagen `registry.access.redhat.com/ubi8/python-38:latest` es una imagen base de Python construida por Red Hat que permite la ejecución de la aplicación con un usuario no privilegiado.
* Con el parámetro `USER` indicamos el usuario sin privilegio que va a ejecutar el servidor web.

Para crear una nueva imagen usamos el comando `buildah build` indicando el nombre con el parámetro `-t`:

```
$ buildah build -t josedom24/app-python
```

Por defecto Buildah busca un fichero `Container` y el contexto en el directorio donde se está ejecutando si queremos indicar un fichero y un contexto en concreto lo tendremos que indicar:

```
$ buildah build -t josedom24/app-python -f Containerfile .
```

Una vez construida la imagen podremos ver que s eha creado, ejecutando:

```
$ buildah images
REPOSITORY                                  TAG           IMAGE ID       CREATED             SIZE
localhost/josedom24/app-python              latest        a7951b30c32a   28 seconds ago      906 MB
```

Y podemos crear un contenedor, en este caso creamos un contenedor rootless cuyo que nos ofrece un servidor web ejecutado con un usuario sin privilegios:

```
$ podman run -d -p 8000:5000 --name python1 josedom24/app-python:latest

$ podman top python1
USER        PID         PPID        %CPU        ELAPSED           TTY         TIME        COMMAND
default     1           0           0.000       29m58.442890345s  ?           0s          python3 app.py 
...
```

