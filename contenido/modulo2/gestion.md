# Gestión de contenedores Docker

## Ciclo de vida de los contenedores

Para ver un ejemplo de los comandos que gestionan el ciclo de vida de un contenedor vamos a ejecutar un contenedor demonio que va escribiendo la hora cada segundo, para ver la salida visualizamos sus logs:

```bash
$ sudo podman run -d --name hora-container ubuntu bash -c 'while true; do echo $(date +"%T"); sleep 1; done'
$ sudo podman logs -f hora-container
```
En otra terminal vamos ejecutando los comandos que nos permiten controlar su ciclo de vida:

* `sudo podman start`: Inicia la ejecución de un contenedor que está parado.
* `sudo podman stop`: Detiene la ejecución de un contenedor en ejecución.
* `sudo podman restart`: Para y vuelve a iniciar la ejecución de un contenedor.
* `sudo podman pause`: Pausa la ejecución de un contenedor.
* `sudo podman unpause`: Continúa la ejecución de un contenedor que estaba pausado..


## Ejecución de comandos en contenedores

Si tenemos un contenedor que está iniciado, podemos ejecutar comandos en él con `podman exec`. En esta ocasión vamos a crear un contenedor que hace algo parecido al anterior, pero en este caso guarda la hora en un fichero cada segundo:

```bash
$ sudo podman run -d --name hora-container2 ubuntu bash -c 'while true; do date +"%T" >> hora.txt; sleep 1; done'
$ sudo podman exec hora-container2 ls
...
hora.txt
...
$ sudo podman exec hora-container2 cat hora.txt
```

## Copiar ficheros en contenedores

Con el comando `podman cp` podemos copiar ficheros a o desde un contenedor. Por ejemplo, si tengo un fichero en mi equipo lo puedo copiar al contenedor:

```bash
$ echo "Curso Podman">podman.txt
$ sudo podman cp podman.txt hora-container2:/tmp
```

Podemos comprobar que el fichero existe en el contenedor:

```bash
$ sudo podman exec hora-container2 cat /tmp/podman.txt
Curso Podman
```

Evidentemente, también podemos copiar ficheros desde el contenedor a nuestro equipo:

```bash
$ sudo podman cp hora-container2:hora.txt .
```

## Visualizar procesos que se ejecutan en un contenedor

Podemos visualizar los procesos que se están ejecutando en un contenedor con el comando `docker top`:

```bash
$ sudo podman top hora-container2
```

## Obtener información de los contenedores

Para obtener información de cualquier objeto Docker vamos a usar el subcomando `inspect`. En el caso de los contenedores, ejecutamos:

```bash
$ sudo podman inspect hora-container2
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

```bash
$ sudo podman inspect --format='{{.Id}}' hora-container2
```

El nombre de la imagen que hemos usado para crear el contenedor:

```bash
$ sudo podman inspect --format='{{.Config.Image}}' hora-container2
```

El valor de las variables de entorno definidas en el contenedor:

```bash
$ sudo podman container inspect -f '{{range .Config.Env}}{{println .}}{{end}}' hora-container2
```

El comando que hemos ejecutado en el contenedor:

```bash
$ sudo podman inspect --format='{{range .Config.Cmd}}{{println .}}{{end}}' hora-container2
```

La dirección IP que tiene el contenedor:

```bash
$ sudo podman inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' hora-container2
```
