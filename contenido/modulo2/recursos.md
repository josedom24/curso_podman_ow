# Limitando los recursos utilizados por un contenedor

Cuando creamos un contenedor, los procesos que se ejecuten en él pueden usar todos los recursos del host en el que se está ejecutando. Puedes limitar los recursos de CPU y memoria al crear un contenedor en Podman utilizando las opciones `--cpus` y `--memory` respectivamente. 

## Limitando el uso de CPU

Lo primero que podemos averiguar es el número de CPUs que tiene el host, para ello ejecutamos el comando:

```
$ nproc --all
```

Para limitar la cantidad de recursos de CPU que puede utilizar un contenedor, puedes utilizar la opción `--cpus`. Puedes especificar la cantidad de CPUs que deseas asignar al contenedor, ya sea en términos de núcleos completos o fracciones de núcleos. Por ejemplo, para limitar un contenedor a utilizar un núcleo completo:

```
$ sudo podman docker run -d --cpus 1 --name servidor_web docker.io/httpd:2.4
```

Podríamos indicar que utilice medio núcleo (`--cpus=0.5`), o usar 2 núcleos (`--cpus=2`) o usar una fracción de cpu, por ejemplo el 75% (`--cpus=0.75`).

Cuando inspeccionas un contenedor con el comando `podman inspect`, puedes utilizar el campo `NanoCpus` para ver la cantidad de CPU asignada al contenedor en unidades de *nanocpus*. Un *nanocpu* es una unidad de medida que representa la fracción de tiempo de CPU.

```
$ sudo podman inspect --format '{{.HostConfig.NanoCpus}}' servidor_web
```

Este comando mostrará la cantidad de CPU asignada al contenedor en unidades de *nanocpus*. Ten en cuenta que la interpretación de estos valores puede no ser intuitiva, pero representan la capacidad de procesamiento relativa asignada al contenedor en comparación con la capacidad total del sistema.

Si deseas obtener información más legible, puedes convertir los *nanocpus* a CPUs utilizando los siguientes comandos:

```
nano_cpus=$(sudo podman inspect --format '{{.HostConfig.NanoCpus}}' servidor_web)
cpus=$(echo "scale=2; $nano_cpus / 1000000000" | bc)
echo "CPUs asignadas al contenedor: $cpus"
```

Este fragmento de código convierte los *nanocpus* a CPUs dividiendo por 1000000000 (mil millones) utilizando la herramienta `bc` (calculadora de línea de comandos). La variable `cpus` contendrá la cantidad de CPUs asignadas al contenedor.


## Limitando el uso de memoria


Para limitar la cantidad de memoria que puede utilizar un contenedor, puedes utilizar la opción `--memory`. Puedes especificar la memoria en bytes, kilobytes, megabytes, gigabytes, o utilizando el formato abreviado con las letras *b, k, m, g*. Por ejemplo para limitar un contenedor a 512 megabytes de memoria:

```
$ sudo podman run -d --memory 512m --name servidor_web docker.io/httpd:2.4
```

Con el comando `podman stats` podemos ver los recursos que está consumiendo un contenedor, y además vemos el límite de RAM que le hemos configurado:

```
$ sudo podman stats servidor_web
ID            NAME          CPU %       MEM USAGE / LIMIT  MEM %       NET IO         BLOCK IO    PIDS        CPU TIME    AVG CPU %
3cab2d116e9f  servidor_web  0.01%       2.716MB / 536.9MB  0.51%       2.04kB / 768B  0B / 0B     82          90.108ms    0.17%
```

También podemos usar `podman inspect` para ver el límite de memoria que hemos configurado en un contenedor:

```
$ sudo podman inspect --format '{{.HostConfig.Memory}}' servidor_web
```

Este comando te dará el límite de memoria en bytes. Si deseas convertirlo a un formato más legible, puedes hacerlo utilizando las siguientes instrucciones en bash:

```
memory_limit=$(sudo podman inspect --format '{{.HostConfig.Memory}}' servidor_web)
memory_limit_mb=$(echo "scale=2; $memory_limit / 1048576" | bc)
echo "Límite de memoria asignado al contenedor: $memory_limit_mb MB"
```

En este fragmento de código, estoy dividiendo el límite de memoria en bytes por 1048576 (que es 1024 al cubo) para convertirlo a megabytes.

Para terminar, indicar que evidentemente podemos limitar la memoria y el número de CPUs utilizadas al mismo tiempo al crear un contenedor:

```
$ sudo podman run -d --memory 512m --cpus 0.5 --name servidor_web docker.io/httpd:2.4
```
