# Introducción a Quadlet

## Systemd

[Systemd](https://systemd.io/) es un sistema de inicio y administración de servicios para sistemas operativos basados en Linux. Además de ser el sistema de inicio que se usa actualmente en las distribuciones Linux, ofrece un conjunto de servicios básicos para el sistema.

Systemd utiliza las **Unidades de Servicios**: Cada servicio o recurso que systemd administra está definido por un **archivo de unidad** (unit file) que especifica cómo debe ser gestionado. Estos archivos pueden configurar el comportamiento del servicio, sus dependencias, el entorno en el que se ejecuta y otras opciones.

## Quadlet

Desde su inicio Podman se ha integrado muy bien con Systemd, posibilitando la gestión de contenedores con unidades se servicios. En un principio se creaban una unidad Systemd que llamaba a podman con el subcomando `run`. Podman también proporcionaba `podman generate systemd` para crear fácilmente dicho archivo Systemd.

Sin embargo, esta opción no es la recomendada, y actualmente se prefiere el uso de Quadlet (que ha sido integrado en Podman) para gestionar la ejecución de contenedores Podman con Systemd.

[Quadlet](https://github.com/containers/quadlet) permite la generación automática de unidades de servicio de Systemd, a partir de unas plantillas que nos permiten definir de manera sencilla el recurso de Podman que queremos controlar con Systemd. Los recursos que actualmente podemos controlar con Quadlet y Systemd son los siguientes:

* Contenedores
* Volúmenes
* Redes
* Pods (En la versión Podman 5)

### ¿Cómo funciona Quadlet?

Quadlet buscará ficheros de plantilla de unidades de sistemas Systemd, en los siguientes directorios:

* Si estamos trabajando con el usuario `root`, los directorio de búsqueda son: `/etc/containers/systemd/` y `/usr/share/containers/systemd/`.
* Si trabajamos con un usuario sin privilegio, el directorio de búsqueda será `$HOME/.config/containers/systemd/`.

Los ficheros de plantilla de unidades Systemd tienen distintas extensiones según el recurso que queremos controlar:

* `.container`: Nos permite definir las características de un contenedor que será gestionado por Systemd ejecutando `podman run`.
* `.volume`: Nos permite definir la definición de volúmenes que serán referenciados en la plantillas del tipo `.container`.
* `.network`: Nos permite definir la definición de redes que serán referenciados en la plantillas del tipo `.container` p `.kube`.
* `.pod`: Nos permite la definición de un Pod que será gestionado por Systemd. En la versión Podman 5.
* `.kube`: Nos permite la definición de escenario creados a parir de ficheros YAML de Kubernetes con la instrucción `podman kube play`.