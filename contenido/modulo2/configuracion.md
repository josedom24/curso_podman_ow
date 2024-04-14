# Configuración de contenedores

## Configuración de contenedores con variables de entorno

Más adelante veremos que al crear un contenedor que necesita alguna configuración específica, lo que vamos a hacer es crear variables de entorno en el contenedor, para que el proceso que inicializa el contenedor pueda realizar dicha configuración.

Para crear una variable de entorno al crear un contenedor usamos el flag `-e` o `--env`:

```
$ podman run -it --name contenedor5 -e USUARIO=prueba ubuntu
root@145b1105bc62:/# echo $USUARIO
prueba
```

# Trabajando con Secrets

Cuando queremos configurar un contenedor y necesitamos enviarle información sensible (contraseñas, claves privadas, ...) podemos usar el recurso **Secret**.

Secret nos permite guarda información sensible  que será codificada o cifrada. Y asegura que dicha información no se guarde en una imagen o en un repositorio de control de versiones.

Un Secret se podrá enviar a un contenedor guardada en un fichero o como valor de una variable de entorno.

## Creación de Secret

Para crear un Secret indicamos su nombre y el valor que va a tener. El valor se puede indicar en el contenido de un fichero o directamente desde la línea de comandos.

Utilizando un fichero, sería de de la siguiente forma:

```
$ echo "esto es un secreto"> secreto.txt
$ sudo podman secret create secreto1 secreto.txt
```

Utilizando la línea de comandos, sería:

```
$ echo "esto es un secreto" | sudo podman secret create secreto2 -
```

Podemos ver los secretos que hemos creado, ejecutando:

```
$ sudo podman secret ls
ID                         NAME        DRIVER      CREATED             UPDATED
4c977053a1c16334311ec4143  secreto1    file        About a minute ago  About a minute ago
5032f047e1b050a009c3de196  secreto2    file        33 seconds ago      33 seconds ago
```

## Información de un Secret

Para obtener información de un Secret, podemos ejecutar:

```
$ sudo podman secret inspect secreto1
[
    {
        "ID": "4c977053a1c16334311ec4143",
        "CreatedAt": "2024-04-08T07:52:34.158449085Z",
        "UpdatedAt": "2024-04-08T07:52:34.158449085Z",
        "Spec": {
            "Name": "secreto1",
            "Driver": {
                "Name": "file",
                "Options": {
                    "path": "/var/lib/containers/storage/secrets/filedriver"
                }
            },
            "Labels": {}
        }
    }
]
```

Donde observamos que el secreto se almacena en un fichero en el directorio `/var/lib/containers/storage/secrets/filedriver` (si estamos usando el usuario `root` para crearlo) que se llama `secretsdata.json`donde el valor del Secret estará codificado en base64.

Si creamos un Secret con nuestro usuario si privilegio el directorio donde se almacena los Secrets será `/$HOME/.local/share/containers/storage/secrets/filedriver`.

## Utilización del Secret en la ejecución de un contenedor

Para enviar el valor de un Secret a un contenedor cuando lo ejecutamos utilizaremos el parámetro `--secret`. El valor del Secret se puede mandar como:

* Una variable de entorno: `--secret <nombre_Secret>,type=env.target=<nombre_variable_entorno>`. Si no se indica el nombre de la variable de entorno, esta se llamará como el Secret.
* El contenido de un fichero que se monta en el contenedor: `--secret <nombre_Secret>,type=mount,target=<path del fichero`. El tipo `mount` es el valor por defecto, además si no se indica el nombre del fichero, el valor por defecto será `/run/secrets/<nombre_Secret>`

Veamos algunos ejemplos:

```
$ sudo podman run --rm --secret secreto1,type=env --name foo alpine printenv secreto1
esto es un secreto
$ sudo podman run --rm --secret secreto1,type=env,target=VARIABLE --name foo alpine printenv VARIABLE
esto es un secreto

$ sudo podman run --rm --secret secreto2 --name foo alpine cat /run/secrets/secreto2
esto es un secreto
$ sudo podman run --rm --secret secreto2,type=mount,target=/opt/secreto.txt --name foo alpine cat /opt/secreto.txt
esto es un secreto
```

## Eliminar un Secret

Para eliminar un Secret que tenemos guardado, sólo tenemos que ejecutar esta instrucción:

```
$ sudo podman secret rm secreto1
```

## Ejemplo: Configuración de un contenedor con la imagen MariaDB


En ocasiones es obligatorio el inicializar alguna variable de entorno para que el contenedor pueda ser ejecutado. Si miramos la [documentación](https://hub.docker.com/_/mariadb) en Docker Hub de la imagen `mariadb`, observamos que podemos definir algunas variables de entorno para la creación y configuración del contenedor (por ejemplo: `MARIADB_DATABASE`,`MARIADB_USER`, `MARIADB_PASSWORD`,...). Pero hay una que la tenemos que indicar de forma obligatoria, la contraseña del usuario `root` (`MARIADB_ROOT_PASSWORD`). Podríamos ejecutar el contenedor indicando directamente la variable de entorno:

```
$ sudo podman run -d --name mimariadb -e MARIADB_ROOT_PASSWORD=my-secret-pw docker.io/mariadb:10.5
```

Pero como la contraseña es una información sensible vamos a guardar dicho valor en un Secret y posteriormente creamos el contenedor:

```
$ echo "my-secret-pw" | sudo podman secret create pass_root -
$ sudo podman run -d --name mimariadb --secret pass_root,type=env,target=MARIADB_ROOT_PASSWORD docker.io/mariadb:10.5
$ sudo podman ps
CONTAINER ID  IMAGE                           COMMAND     CREATED        STATUS        PORTS       NAMES
6c1488d6cb4a  docker.io/library/mariadb:10.5  mysqld      8 seconds ago  Up 7 seconds              mimariadb
```

Podemos ver que se ha creado una variable de entorno:

```
$ sudo podman exec -it mimariadb env
...
MARIADB_ROOT_PASSWORD=my-secret-pw
...
```

Y para acceder podemos ejecutar:

```
$ sudo podman exec -it mimariadb bash
root@9c3effd891e3:/# mysql -u root -p"$MARIADB_ROOT_PASSWORD" 
...

MariaDB [(none)]> 
```
Otra forma de hacerlo sería:

```
$ sudo podman exec -it mimariadb mysql -u root -p -h 127.0.0.1
Enter password: 
...
MariaDB [(none)]> 
```

## Accediendo a servidor de base de datos desde el exterior

En el ejemplo anterior hemos accedido a la base de datos de dos formas: 

1. Ejecutado un comando `bash` para acceder al contenedor y desde dentro hemos utilizado el cliente de MariaDB para acceder a la base de datos.
2. Ejecutando directamente en el contenedor el cliente de MariaDB.

En esta ocasión vamos a mapear los puertos para acceder desde el exterior a la base de datos:

Lo primero que vamos a hacer es eliminar el contenedor anterior:

``` 
$ sudo podman rm -f mimariadb
```

Y a continuación vamos a crear otro contenedor, pero en esta ocasión vamos a mapear el puerto 3306/tcp del Host Docker con el puerto 3306/tcp del contenedor:

``` 
$ sudo podman run -d -p 3306:3306 --name mimariadb -e MARIADB_ROOT_PASSWORD=my-secret-pw docker.io/mariadb:10.5
```

Comprobamos que los puertos se han mapeado y que el contenedor está ejecutándose:

```
$ sudo podman ps
CONTAINER ID  IMAGE                           COMMAND     CREATED        STATUS        PORTS                   NAMES
ee15342ca308  docker.io/library/mariadb:10.5  mysqld      3 seconds ago  Up 3 seconds  0.0.0.0:3306->3306/tcp  mimariadb
```

Ahora desde nuestro equipo, donde hemos instalado un cliente de MariaDB (`sudo apt install mariadb-client` en Debian/ubuntu o `sudo dnf install community-mysql.x86_64` en Fedora), nos conectamos al host:

```
$ mysql -u root -p -h 127.0.0.1
Enter password: 
...
mysql> 
```

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
