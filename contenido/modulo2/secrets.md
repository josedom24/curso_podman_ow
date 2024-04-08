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