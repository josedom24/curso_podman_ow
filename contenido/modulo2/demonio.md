# Ejecución de contenedores demonios

En esta ocasión hemos utilizado la opción `-d` o `--detach` del comando `podman run`, para que la ejecución del comando en el contenedor se haga en segundo plano, de manera desatendida, sin estar conectada a la entrada y salida estándar.

Por ejemplo, en este ejemplo ejecutamos un proceso demonio en el contenedor (proceso que se ejecuta indefinidamente) y utilizamos la poción `-d` para que se ejecute de manera desatendida, en segundo planto. En el ejemplo ejecutamos un bucle infinito que va escribiendo secuencialmente números en la salida estándar y en un fichero cada un segundo:

```
$ podman run -d --name contenedor1 ubuntu bash -c 'i=0;while true; do echo $((i=i+1)); echo $i >> numeros.txt; sleep 1; done'
```

En la instrucción `podman run` hemos ejecutado el comando con `bash -c` que nos permite ejecutar uno o más comandos en el contenedor de forma más compleja (por ejemplo, indicando ficheros dentro del contenedor).

Comprobamos que el contenedor se está ejecutando:

```
$ podman ps
CONTAINER ID  IMAGE                            COMMAND               CREATED        STATUS        PORTS       NAMES
7a3e97be08c1  docker.io/library/ubuntu:latest  bash -c i=0;while...  3 minutes ago  Up 3 minutes              contenedor1
```

Podemos visualizar los logs del contenedor, ejecutando el siguiente comando:

```
$ sudo podman logs contenedor1
```

Con la opción `logs -f` seguimos visualizando los logs en tiempo real.

## Ciclo de vida de los contenedores

Tenemos varios comandos que nos permiten controlar el ciclo de vida del contenedor. En una terminal podemos visualizar los logs del contenedor y en otro terminal ejecutar los comandos para ver cómo se comporta el contenedor:

* `podman start`: Inicia la ejecución de un contenedor que está parado.
* `podman stop`: Detiene la ejecución de un contenedor en ejecución.
* `podman restart`: Para y vuelve a iniciar la ejecución de un contenedor.
* `podman pause`: Pausa la ejecución de un contenedor.
* `podman unpause`: Continúa la ejecución de un contenedor que estaba pausado..

## Ejecución de comandos en contenedores

Si tenemos un contenedor que está iniciado, podemos ejecutar comandos en él con `podman exec`. Por ejemplo:

```
$ podman exec contenedor1 ls
...
numeros.txt
...
$ podman exec contenedor1 cat numeros.txt
```

Si queremos acceder interactivamente al contenedor, podemos ejecutar:

```
$ podman exec -it contenedor1 bash
```

## Copiar ficheros en contenedores

Con el comando `podman cp` podemos copiar ficheros a o desde un contenedor. Por ejemplo, si tengo un fichero en mi equipo lo puedo copiar al contenedor:

```
$ echo "Curso Podman">podman.txt
$ podman cp podman.txt contenedor1:/tmp
```

Podemos comprobar que el fichero existe en el contenedor:

```
$ podman exec contenedor1 cat /tmp/podman.txt
Curso Podman
```

Evidentemente, también podemos copiar ficheros desde el contenedor a nuestro equipo:

```
$ podman cp contenedor1:numeros.txt .
```

## Eliminar un contenedor demonio

Hay que tener en cuenta que un contenedor que esta ejecutándose no puede ser eliminado. Tendríamos que parar el contenedor y posteriormente borrarlo:

```
$ podman stop contenedor1
$ podman rm contenedor1
```

Otra opción es borrarlo a la fuerza:

```
$ podman rm -f contenedor1
```

Para eliminar todos los contenedores que están parados:

```
$ podman container prune
WARNING! This will remove all non running containers.
Are you sure you want to continue? [y/N] 
```

Otra solución sería:

```
$ podman rm --all
```

Finalmente para eliminar todos los contenedores:

```
$ podman rm -f --all
```
