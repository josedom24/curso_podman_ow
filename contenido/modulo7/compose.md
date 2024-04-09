# Creando escenarios multicontenedor con Compose

Como hemos visto hasta ahora, en muchas ocasiones necesitamos correr varios contenedores para que nuestra aplicación funcione. En cualquiera de estos casos es necesario tener varios contenedores:

* Si tenemos **aplicaciones monolíticas**, vamos a usar un esquema **multicapa**. Necesitamos **varios servicios** para que la aplicación funcione. Partiendo del principio de que cada contenedor ejecuta un sólo proceso, si necesitamos que la aplicación use varios servicios (web, base de datos, proxy inverso, ...) cada uno de ellos se implementará en un contenedor.
* Si tenemos construida nuestra aplicación con **microservicios**, cada uno de ellos se podrá implementar en un contenedor independiente.

Cuando trabajamos con escenarios donde necesitamos correr varios contenedores podemos utilizar [Compose](https://compose-spec.io/). Compose nos ofrece una especificación para gestionar escenarios multicontenedor. Vamos a definir el escenario en un fichero llamado `compose.yaml` y vamos a gestionar el ciclo de vida de la aplicación y de todos los contenedores con la herramienta `podman-compose`.

El fichero de definición del escenario también puede ser llamado `compose.yml`, y por compatibilidad con versiones antiguas se puede llamar `docker-compose.yaml` o `docker-compose.yml`.

## podman-compose

[podman-compose](https://github.com/containers/podman-compose) es una aplicación que implementa la especificación Compose, y que permite la gestión de escenarios multicontenedor en Podman.

`podman-compose` es un script escrito en python, por lo que podemos hacer la instalación:

* Utilizando el paquete oficial de la distribución: `apt install podman-compose`, `dfn install podman-compose`.
* O utilizando `pip`: `pip3 install podman-compose`.
* Coe puede usar también desde Podman Desktop.

Una vez instalado, podremos utilizar directamente el ejecutable de `podman-compose` o una utilidad que nos ofrece el cliente de Podman: `podman compose`.

## Ventajas de usar Compose

* Hacer todo de manera **declarativa** para que no tenga que repetir todo el proceso cada vez que construyo el escenario.
* Los archivos de declaración de Compose se pueden **distribuir** de manera sencilla, aumentando la colaboración entre los equipos de desarrollo.
* Poner en funcionamiento todos los contenedores que necesita mi aplicación de una sola vez y debidamente configurados.
* Garantizar que los contenedores **se arrancan en el orden adecuado**. Por ejemplo: mi aplicación no podrá funcionar debidamente hasta que no esté el servidor de bases de datos funcionando en marcha.
* Asegurarnos de que hay **comunicación** entre los contenedores que pertenecen a la aplicación.
* **Portabilidad entre entornos**: Compose admite variables en el archivo de declaración. Puede utilizar estas variables para personalizar su composición para diferentes entornos o diferentes usuarios.