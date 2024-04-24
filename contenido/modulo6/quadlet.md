# Introducción a Quadlet

## Systemd

[Systemd](https://systemd.io/) es un sistema de inicio y administración de servicios para sistemas operativos basados en Linux. Además de ser el sistema de inicio que se usa actualmente en las distribuciones Linux, ofrece un conjunto de servicios básicos para el sistema.

Systemd utiliza las **Unidades de Servicios**: Cada servicio o recurso que Systemd administra está definido por un **archivo de unidad** (unit file) que especifica cómo debe ser gestionado. Estos archivos pueden configurar el comportamiento del servicio, sus dependencias, el entorno en el que se ejecuta y otras opciones.

## Política de reinicio de los contenedores Podman

Un inconveniente de que Podman no utilice un demonio que controla la ejecución de los contenedores, es que si reiniciamos el host, los contenedores no se inician.

Una posible solución es activar el servicio `podman-restart` que reinicia los contenedores cuya política de reinicio esté activa, con el parámetro `--restart=always` de `podman run`. Por ejemplo:

```
$ sudo podman run -d --name c1 --restart=always quay.io/libpod/banner
$ podman run -d --name c2 --restart=always quay.io/libpod/banner
```

Para activar el servicio:

* En entorno rootful:
  ```
  $ sudo systemctl enable podman-restart
  $ sudo systemctl start podman-restart
  ```
* En entorno rootless:
  ```
  $ systemctl --user enable podman-restart
  $ systemctl --user start podman-restart
  ```
Ahora puedes comprobar que los contenedores se inician de forma automática tras el reinicio del host.

Otra solución al inicio automático de los contenedores después de un reinicio sería integrar la ejecución de contenedores con Systemd. Para conseguir este objetivo vamos a usar Quadlet.

## Quadlet

Desde su inicio Podman se ha integrado muy bien con Systemd, posibilitando la gestión de contenedores con unidades se servicios. En un principio se creaban una unidad Systemd que llamaba a Podman con el subcomando `run`. Podman también proporcionaba `podman generate systemd` para crear fácilmente dicho archivo Systemd.

Sin embargo, esta opción no es la recomendada, y actualmente se prefiere el uso de Quadlet (que ha sido integrado en Podman) para gestionar la ejecución de contenedores Podman con Systemd.

[Quadlet](https://github.com/containers/quadlet) permite la generación automática de unidades de servicio de Systemd, a partir de unas plantillas que nos permiten definir de manera sencilla el recurso de Podman que queremos controlar con Systemd. Los recursos que actualmente podemos controlar con Quadlet y Systemd son los siguientes:

* Contenedores
* Volúmenes
* Redes
* Pods 

### ¿Cómo funciona Quadlet?

Quadlet buscará ficheros de plantilla de unidades de sistemas Systemd, en los siguientes directorios:

* Si estamos trabajando con el usuario `root`, los directorio de búsqueda son: `/etc/containers/systemd/` y `/usr/share/containers/systemd/`.
* Si trabajamos con un usuario sin privilegios, el directorio de búsqueda será `$HOME/.config/containers/systemd/`.

Los ficheros de plantilla de unidades Systemd tienen distintas extensiones según el recurso que queremos controlar:

* `.container`: Nos permite definir las características de un contenedor que será gestionado por Systemd ejecutando `podman run`.
* `.volume`: Nos permite definir la definición de volúmenes que serán referenciados en la plantillas del tipo `.container`.
* `.network`: Nos permite definir la definición de redes que serán referenciados en la plantillas del tipo `.container` y `.kube`.
* `.pod`: Nos permite la definición de un Pod que será gestionado por Systemd. Sólo funciona en la versión Podman 5.
* `.kube`: Nos permite la definición de escenario creados a parir de ficheros YAML de Kubernetes con la instrucción `podman kube play`.
