# Gestión de Pods

La gestión de los Pods la realizaremos utilizando el comando `podman pod`. En este apartado crearemos Pods con el usuario privilegiado por lo que vamos a usar `sudo` para la ejecución de los comandos.

## Creación de un Pod

Para crear un Pod, usamos el comando `podman pod create`, por ejemplo para crear un Pod llamado `pod1`, ejecutamos:

```
$ sudo podman pod create --name pod1
```

Podemos visualizar el Pod que hemos creado:

```
$ sudo podman pod ps
POD ID        NAME        STATUS      CREATED        INFRA ID      # OF CONTAINERS
45d3c5f01747  pod1        Created     8 seconds ago  41f153d29f02  1
```
Como vemos esta compuesto por un contenedor, que será el contenedor **infra**. Si queremos mostrar los nombres de los contenedores de los que está formado, usamos el parámetro `--ctr-names`:

```
$ sudo podman pod ps --ctr-names
POD ID        NAME        STATUS      CREATED         INFRA ID      NAMES
45d3c5f01747  pod1        Created     32 seconds ago  41f153d29f02  45d3c5f01747-infra
```

## Añadir contenedores a un Pod

Para añadir un contenedor a un Pod, a la hora de crearlo debemos indicar el nombre del Pod:

```
$ sudo podman run -dt --pod pod1 --name principal alpine:latest top
```

Al iniciar el contenedor principal, el Pod se ha iniciado:

```
$ sudo podman pod ps
POD ID        NAME        STATUS      CREATED        INFRA ID      # OF CONTAINERS
45d3c5f01747  pod1        Running     8 minutes ago  41f153d29f02  2
```

Comprobamos que tenemos dos contenedores en el Pod. Podemos visualizar los contenedores con la información del Pod al que pertenecen, ejecutando:

```
$ sudo podman ps --pod
CONTAINER ID  IMAGE                                    COMMAND     CREATED        STATUS        PORTS       NAMES               POD ID        PODNAME
41f153d29f02  localhost/podman-pause:4.9.4-1711445992              9 minutes ago  Up 3 minutes              45d3c5f01747-infra  45d3c5f01747  pod1
dde078d7cf4e  docker.io/library/alpine:latest          top         3 minutes ago  Up 3 minutes              principal           45d3c5f01747  pod1
```

## Ciclo de vida de un Pod

Aunque podemos controlar el ciclo de vida independientemente de cada contenedor, al operar sobre el Pod estaremos actuando sobre todos los contenedores. Por ejemplo, para pausar y continuar los contenedores de un Pod:

```
$ sudo podman pod pause pod1
$ sudo podman pod unpause pod1
```

Para parar e iniciar todos los contenedores:

```
$ sudo podman pod stop pod1
$ sudo podman pod start pod1
```

Para reiniciar los contenedores de un Pod:

```
$ sudo podman pod restart pod1
```

## Obteniendo información de un Pod

Podemos obtener la información detallada de un Pod ejecutando:

```
$ sudo podman pod inspect pod1
```

Si tenemos un sólo contenedor princiapl en el Pod podemos ver los logs de salida, ejecutando:

```
$ sudo podman pod logs pod1
```

Sin embargo, si añadimos un segunado contenedor:

```
$ sudo podman run -dt --pod pod1 --name secundario alpine:latest top

$ sudo podman ps  --pod
CONTAINER ID  IMAGE                                    COMMAND     CREATED             STATUS             PORTS       NAMES               POD ID        PODNAME
41f153d29f02  localhost/podman-pause:4.9.4-1711445992              19 minutes ago      Up 13 minutes                  45d3c5f01747-infra  45d3c5f01747  pod1
dde078d7cf4e  docker.io/library/alpine:latest          top         13 minutes ago      Up 13 minutes                  primario            45d3c5f01747  pod1
b220cb7dfdf1  docker.io/library/alpine:latest          top         About a minute ago  Up About a minute              secundario          45d3c5f01747  pod1
```

Para obtener los logs hay que especificar el contenedor con el parámetro `-c`:

```
$ sudo podman pod logs -c secundario pod1
```

Finalmente para ver los procesos que se están ejecutando en los distintos contenedores:

```
$ sudo podman pod top pod1
USER        PID         PPID        %CPU        ELAPSED           TTY         TIME        COMMAND
0           1           0           0.000       14m54.557309601s  ?           0s          /catatonit -P 
root        1           0           0.000       14m54.559080387s  pts/0       0s          top 
root        1           0           0.000       2m55.562434639s   pts/0       0s          top 
```

## Eliminar los Pods

Al borrar un Pod, se eliminaran todos los contenedores que forman parte de él. Si queremos borrar un Pod debe estar parado:

```
$ sudo podman pod stop pod1
$ sudo podman pod rm pod1
```

Si queremos forzar la eliminación de un Pod aunque este en ejecución:

```
sudo podman pod rm -f pod1
```

Si queremos borrar todos los Pods parados y en ejecución:

```
sudo podman pod rm --all
sudo podman pod rm -f --all
```

Por último para eliminar los Pods que están parados:

``` 
sudo podman pod prune
WARNING! This will remove all stopped/exited pods..
Are you sure you want to continue? [y/N]
```
