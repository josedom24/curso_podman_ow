# Gestión de contenedores Docker

## Ciclo de vida de los contenedores

Para ver un ejemplo de los comandos que gestionan el ciclo de vida de un contenedor vamos a ejecutar un contenedor demonio que va escribiendo la hora cada segundo, para ver la salida visualizamos sus logs:

```
$ podman run -d --name hora-container ubuntu bash -c 'while true; do echo $(date +"%T"); sleep 1; done'
$ podman logs -f hora-container
```
En otra terminal vamos ejecutando los comandos que nos permiten controlar su ciclo de vida:

* `podman start`: Inicia la ejecución de un contenedor que está parado.
* `podman stop`: Detiene la ejecución de un contenedor en ejecución.
* `podman restart`: Para y vuelve a iniciar la ejecución de un contenedor.
* `podman pause`: Pausa la ejecución de un contenedor.
* `podman unpause`: Continúa la ejecución de un contenedor que estaba pausado..


## Ejecución de comandos en contenedores

Si tenemos un contenedor que está iniciado, podemos ejecutar comandos en él con `podman exec`. En esta ocasión vamos a crear un contenedor que hace algo parecido al anterior, pero en este caso guarda la hora en un fichero cada segundo:

```
$ podman run -d --name hora-container2 ubuntu bash -c 'while true; do date +"%T" >> hora.txt; sleep 1; done'
$ podman exec hora-container2 ls
...
hora.txt
...
$ podman exec hora-container2 cat hora.txt
```

## Copiar ficheros en contenedores

Con el comando `podman cp` podemos copiar ficheros a o desde un contenedor. Por ejemplo, si tengo un fichero en mi equipo lo puedo copiar al contenedor:

```
$ echo "Curso Podman">podman.txt
$ podman cp podman.txt hora-container2:/tmp
```

Podemos comprobar que el fichero existe en el contenedor:

```
$ podman exec hora-container2 cat /tmp/podman.txt
Curso Podman
```

Evidentemente, también podemos copiar ficheros desde el contenedor a nuestro equipo:

```
$ podman cp hora-container2:hora.txt .
```

## Visualizar procesos que se ejecutan en un contenedor

Podemos visualizar los procesos que se están ejecutando en un contenedor con el comando `docker top`:

```
$ podman top hora-container2
```

## Obtener información de los contenedores

Para obtener información de cualquier objeto Docker vamos a usar el subcomando `inspect`. En el caso de los contenedores, ejecutamos:

```
$ podman inspect hora-container2
```
Nos muestra mucha información en formato JSON (JavaScript Object Notation) y nos da datos sobre aspectos como:

* El id del contenedor.
* Los puertos abiertos y sus redirecciones.
* Los bind mounts y volúmenes usados.
* El tamaño del contenedor (si ejecutamos `podman inspect -s`).
* La configuración de red del contenedor.
* El comando que se esta ejecutando en el contenedor.
* El valor de las variables de entorno.
* Y muchas más cosas....

Como nos devuelve mucha información podemos filtrar los campos que nos interesan, por ejemplo:

El identificado del contenedor:

```
$ podman inspect --format='{{.Id}}' hora-container2
```

El nombre de la imagen que hemos usado para crear el contenedor:

```
$ podman inspect --format='{{.Config.Image}}' hora-container2
```

El valor de las variables de entorno definidas en el contenedor:

```
$ podman container inspect --format '{{range .Config.Env}}{{println .}}{{end}}' hora-container2
```

El comando que hemos ejecutado en el contenedor:

```
$ podman inspect --format='{{range .Config.Cmd}}{{println .}}{{end}}' hora-container2
```

La dirección IP que tiene el contenedor:

```
$ podman inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' hora-container2
```

## Etiquetando los contenedores con Labels

Las etiquetas son un mecanismo para guardar metadatos en los objetos Docker: en contenedores, imágenes, volúmenes, redes, etc. Podemos utilizar etiquetas para organizar los distintos objetos Docker que estemos utilizando.

Una etiqueta es una información del tipo clave-valor, almacenado como una cadena. Para etiquetar un contenedor en su creación utilizaremos el parámetro `-l` (`--label`).

```
$ podman run -l servicio=bd -l entorno=produccion --name contenedor ubuntu
```

Hay que tener en cuenta que estos contenedores tendrán además de las etiquetas indicadas en su creación, las etiquetas que estén definidas en la imagen que hemos utilizado para su creación.

```
$ podman inspect --format '{{range $key, $value := .Config.Labels}}{{$key}}: {{$value}}{{"\n"}}{{end}}' contenedor
entorno: produccion
org.opencontainers.image.ref.name: ubuntu
org.opencontainers.image.version: 22.04
servicio: bd
```

A la hora de listar los contenedores podemos filtrar por varios criterios, entre ellos podemos usar las etiquetas para hacer el filtro.

Mostrar los contenedores que tienen la etiqueta entorno con el valor `produccion`:

```
podman ps -a --filter="label=entorno=produccion"
```


