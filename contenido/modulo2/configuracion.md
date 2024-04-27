# Configuración de contenedores

## Configuración de contenedores con variables de entorno

Más adelante veremos que al crear un contenedor que necesita alguna configuración específica, lo que vamos a hacer es crear variables de entorno en el contenedor, para que el proceso que inicializa el contenedor pueda realizar dicha configuración.

Para crear una variable de entorno al crear un contenedor usamos el parámetro `-e` o `--env`:

```
$ podman run -it --name contenedor2 -e USUARIO=prueba ubuntu
root@145b1105bc62:/# echo $USUARIO
prueba
```

## Trabajando con Secrets

Cuando queremos configurar un contenedor y necesitamos enviarle información sensible (contraseñas, claves privadas, ...) podemos usar el recurso **Secret**.

Secret nos permite guarda información sensible que será codificada o cifrada. Se asegura que dicha información no se guarde en una imagen o en un repositorio de control de versiones.

Un Secret se podrá enviar a un contenedor guardada en un fichero o como valor de una variable de entorno.

### Creación de Secret

Para crear un Secret indicamos su nombre y el valor que va a tener. El valor se puede indicar en el contenido de un fichero o directamente desde la línea de comandos.

Utilizando un fichero, sería de de la siguiente forma:

```
$ echo "esto es un secreto"> secreto.txt
$ podman secret create secreto1 secreto.txt
```

Utilizando la línea de comandos, sería:

```
$ echo "esto es un secreto" | podman secret create secreto2 -
```

Podemos ver los secretos que hemos creado, ejecutando:

```
$ podman secret ls
ID                         NAME        DRIVER      CREATED             UPDATED
4c977053a1c16334311ec4143  secreto1    file        About a minute ago  About a minute ago
5032f047e1b050a009c3de196  secreto2    file        33 seconds ago      33 seconds ago
```

### Información de un Secret

Para obtener información de un Secret, podemos ejecutar:

```
$ podman secret inspect secreto1
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
                    "path": "/home/usuario/.local/share/containers/storage/secrets/filedriver"
                }
            },
            "Labels": {}
        }
    }
]
```

* Si creamos con un usuario sin privilegios el secreto, se almacenará en un fichero en el directorio `/$HOME/.local/share/containers/storage/secrets/`.
* Si creamos con el usuario `root` el secreto, se almacenará en el directorio `/var/lib/containers/storage/secrets/filedriver`.
* El fichero donde se almacena se llama `secretsdata.json`y su contenido será el valor del secreto codificado en base64.

### Utilización del Secret en la ejecución de un contenedor

Para enviar el valor de un Secret a un contenedor cuando lo ejecutamos utilizaremos el parámetro `--secret`. El valor del Secret se puede mandar como:

* Una variable de entorno: `--secret <nombre_Secret>,type=env,target=<nombre_variable_entorno>`. Si no se indica el nombre de la variable de entorno, esta se llamará como el Secret.
* El contenido de un fichero que se monta en el contenedor: `--secret <nombre_Secret>,type=mount,target=<path del fichero`. El tipo `mount` es el valor por defecto, además si no se indica el nombre del fichero, el valor por defecto será `/run/secrets/<nombre_Secret>`

Veamos algunos ejemplos:

```
$ podman run --rm --secret secreto1,type=env alpine printenv secreto1
esto es un secreto
$ podman run --rm --secret secreto1,type=env,target=VARIABLE alpine printenv VARIABLE
esto es un secreto

$ podman run --rm --secret secreto2 alpine cat /run/secrets/secreto2
esto es un secreto
$ podman run --rm --secret secreto2,type=mount,target=/opt/secreto.txt alpine cat /opt/secreto.txt
esto es un secreto
```

### Eliminar un Secret

Para eliminar un Secret que tenemos guardado, sólo tenemos que ejecutar esta instrucción:

```
$ podman secret rm secreto1
```

## Limitando los recursos utilizados por un contenedor

Cuando creamos un contenedor, los procesos que se ejecuten en él pueden usar todos los recursos del host en el que se está ejecutando. Puedes limitar los recursos de CPU y memoria al crear un contenedor en Podman utilizando las opciones `--cpus` y `--memory` respectivamente:

```
$ podman run -d --memory 512m --cpus 0.5 --name servidor_web -p 8082:80 docker.io/httpd:2.4
```

Para limitar la cantidad de recursos de CPU que puede utilizar un contenedor, puedes utilizar la opción `--cpus`. Puedes especificar la cantidad de CPUs que deseas asignar al contenedor, ya sea en términos de núcleos completos o fracciones de núcleos: podríamos indicar que utilice medio núcleo (`--cpus=0.5`), o usar 2 núcleos (`--cpus=2`) o usar una fracción de cpu, por ejemplo el 75% (`--cpus=0.75`).

Cuando inspeccionas un contenedor con el comando `podman inspect`, puedes utilizar el campo `NanoCpus` para ver la cantidad de CPU asignada al contenedor en unidades de *nanocpus*. Un *nanocpu* es una unidad de medida que representa la fracción de tiempo de CPU.

```
$ podman inspect --format '{{.HostConfig.NanoCpus}}' servidor_web
```

Para limitar la cantidad de memoria que puede utilizar un contenedor, puedes utilizar la opción `--memory`. Puedes especificar la memoria en bytes, kilobytes, megabytes, gigabytes, o utilizando el formato abreviado con las letras *b, k, m, g*. Por ejemplo para limitar un contenedor a 512 megabytes de memoria:

Con el comando `podman stats` podemos ver los recursos que está consumiendo un contenedor, y además vemos el límite de RAM que le hemos configurado:

```
$ sudo podman stats servidor_web
ID            NAME          CPU %       MEM USAGE / LIMIT  MEM %       NET IO         BLOCK IO    PIDS        CPU TIME    AVG CPU %
3cab2d116e9f  servidor_web  0.01%       2.716MB / 536.9MB  0.51%       2.04kB / 768B  0B / 0B     82          90.108ms    0.17%
```

También podemos usar `podman inspect` para ver el límite de memoria (en bytes) que hemos configurado en un contenedor:

```
$ sudo podman inspect --format '{{.HostConfig.Memory}}' servidor_web
```
