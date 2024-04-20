# Creando escenarios multicontenedor con Compose

Como hemos visto hasta ahora, en muchas ocasiones necesitamos correr varios contenedores para que nuestra aplicación funcione. En cualquiera de estos casos es necesario tener varios contenedores:

* Si tenemos **aplicaciones monolíticas**, vamos a usar un esquema **multicapa**. Necesitamos **varios servicios** para que la aplicación funcione. Partiendo del principio de que cada contenedor ejecuta un sólo proceso, si necesitamos que la aplicación use varios servicios (web, base de datos, proxy inverso, ...) cada uno de ellos se implementará en un contenedor.
* Si tenemos construida nuestra aplicación con **microservicios**, cada uno de ellos se podrá implementar en un contenedor independiente.

Cuando trabajamos con escenarios donde necesitamos correr varios contenedores podemos utilizar [Compose](https://compose-spec.io/). Compose nos ofrece una especificación para gestionar escenarios multicontenedor. Vamos a definir el escenario en un fichero llamado `compose.yaml` y vamos a gestionar el ciclo de vida de la aplicación y de todos los contenedores con la herramienta `podman-compose`.

El fichero de definición del escenario también puede ser llamado `compose.yml`, y por compatibilidad con versiones antiguas se puede llamar `docker-compose.yaml` o `docker-compose.yml`.

## podman-compose

[podman-compose](https://github.com/containers/podman-compose) es una aplicación que implementa la especificación Compose, y que permite la gestión de escenarios multicontenedor en Podman.

`podman-compose` es un script escrito en python, por lo que podemos hacer la instalación:

* Utilizando el paquete oficial de la distribución: `apt install podman-compose`, `dnf install podman-compose`.
* O utilizando `pip`: `pip3 install podman-compose`.
* Coe puede usar también desde Podman Desktop.

Una vez instalado, podremos utilizar directamente el ejecutable de `podman-compose` o una utilidad que nos ofrece el cliente de Podman: `podman compose`.

## docker-compose

`docker-compose` es un script escrito en Python que tradicionalmente se ha utilizado para crear escenarios multiescenario en Docker. Actualmente la funcionalidad para trabajar con Compose se ha añadido al cliente Docker y podemos usar el comando `docker compose` para trabajar.

Como alternativa al uso de `podman-compose` podemos usar `docker-compose` para ello hay que instalar los siguiente paquetes:

En distribuciones Debian/Ubuntu:

```
$ sudo apt install docker-compose podman-docker
```

El la distribución Fedora:

```
$ sudo dnf install docker-compose podman-docker 
```

El paquete `podman-docker` añade un alias para que al poner el comando `docker` se ejecute el comando `podman`. Por último necesitamos activar el socket de Podman para que `docker-compose` se pueda conectar a la API de Podman. 

Si queremos usar `docker-compose` en entorno rootful:

```
$ sudo systemctl start podman.socket
```

Si queremos usarlo en entornos rootless:

```
$ systemctl --user start podman.socket
$ export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/podman/podman.sock
```

## Ventajas de usar Compose

* Hacer todo de manera **declarativa** para que no tenga que repetir todo el proceso cada vez que construyo el escenario.
* Los archivos de declaración de Compose se pueden **distribuir** de manera sencilla, aumentando la colaboración entre los equipos de desarrollo.
* Poner en funcionamiento todos los contenedores que necesita mi aplicación de una sola vez y debidamente configurados.
* Garantizar que los contenedores **se arrancan en el orden adecuado**. Por ejemplo: mi aplicación no podrá funcionar debidamente hasta que no esté el servidor de bases de datos funcionando en marcha.
* Asegurarnos de que hay **comunicación** entre los contenedores que pertenecen a la aplicación.
* **Portabilidad entre entornos**: Compose admite variables en el archivo de declaración. Puede utilizar estas variables para personalizar su composición para diferentes entornos o diferentes usuarios.