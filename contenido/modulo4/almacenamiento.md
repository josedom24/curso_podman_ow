# Almacenamiento en Podman

Cuando se elimina un contenedor, la capa del contenedor de lectura y escritura también se elimina, por lo que podemos afirmar que los **contenedores son efímeros**. Sus datos se pierden al ser eliminados.

Podman nos proporciona varias soluciones para persistir los datos de los contenedores:

* **Volúmenes Podman**, directorios creados por Podman que podemos montar en el contenedor.
* Los **bind mount**, montaje de un directorio o archivo desde el host en el contenedor.
* Los **tmpfs mounts**, montaje de un sistema de archivos en memoria temporal (tmpfs) en el contenedor.
* Los **glob mounts**, montaje de varios archivos o directorios que coinciden con un patrón dentro del contenedor. Es útil cuando se necesita montar múltiples archivos o directorios que siguen un patrón específico.
* Montaje de **imágenes**, montaje de una capa de imagen del contenedor.
* Los **devpts mounts**, montaje de un sistema de archivos devpts, que proporciona acceso a pseudo-terminales, en el contenedor.

## Volúmenes Podman

* Los volúmenes son creados y gestionados por Podman.
* Un volumen corresponde a un directorio en el host, por tanto, la información se almacena en una parte del sistema de ficheros que es gestionada por Podman.
    * Si lo creamos con el usuario `root`: `/var/lib/containers/storage/volumes`.
    * Si lo creamos con un usuario sin privilegios: `$HOME/.local/share/containers/storage/volumes`.
* Cuando se usa un volumen en un contenedor, el directorio correspondiente se monta en el sistema de archivo del contenedor.
* Los procesos ajenos a Podman no deben modificar esta parte del sistema de archivos.
* Un volumen dado puede ser montado en múltiples contenedores simultáneamente. 
* Cuando ningún contenedor en ejecución está utilizando un volumen, el volumen sigue estando disponible para Podman y no se elimina automáticamente. 
* Cuando creas un volumen, puede tener nombre o ser anónimo. 
* A los volúmenes anónimos se les da un nombre aleatorio que se garantiza que sea único dentro del host.
* La gestión de los volúmenes se hace con el comando `podman volume`.
* Cuando usar los volúmenes:
    * Compartir datos entre múltiples contenedores en ejecución.
    * Cuando no se garantiza que el host tenga una determinada estructura de directorios o archivos. Los volúmenes te ayudan a desacoplar la configuración del host del tiempo de ejecución del contenedor.
    * Cuando desea almacenar los datos de su contenedor en un servidor remoto o en un proveedor de nube.
    * Cuando necesites hacer copias de seguridad, restaurar o migrar datos de un host a otro, los volúmenes son una mejor opción, ya que simplemente debes copiar el directorio donde se guardan los volúmenes.


## Bind Mount

* Nos permite que un archivo o directorio del host se monte en un contenedor.
* El archivo o directorio es referenciado por su ruta completa en el host.
* No es necesario que el archivo o directorio ya exista en el host. Se crea bajo demanda si aún no existe.
* No puedes utilizar el cliente Podman para gestionarlos.
* Al realizar cambios sobre los ficheros del bind mount en el anfitrión, se cambian directamente en el contenedor.
* Cuando usar bind mount:
    * Compartir archivos de configuración desde la máquina anfitriona a los contenedores.
    * Compartir código fuente o artefactos de construcción entre un entorno de desarrollo en el host y un contenedor.
    * Cuando hay necesidad de que otras aplicaciones que no sean Podman tengan acceso a esos ficheros, ya sean código, ficheros etc...


## Aspectos a tener en cuenta en el uso de volúmenes o bind mount

* Si montas un volumen vacío en un directorio del contenedor en el que existen archivos o directorios, estos archivos o directorios se propagan (copian) al volumen. 
* Del mismo modo, si inicias un contenedor y especificas un volumen que aún no existe, se creará un volumen vacío para ti. 
* Si montas un bind mount o un volumen no vacío en un directorio del contenedor en el que existen algunos archivos o directorios, estos archivos o directorios quedan ocultos por el montaje.

## Almacenamiento y SELinux

SELinux, que significa "Security-Enhanced Linux" (Linux con seguridad mejorada), es una función de seguridad para sistemas operativos Linux que proporciona un control avanzado sobre los permisos de acceso del sistema. 

Cuando trabajamos con Podman en un sistema operativo en que SELinux está activo, tenemos que tener en cuenta algunas consideraciones a la hora de montar directorios en los contenedores.

SELinux configura ciertos directorios de manera adecuada para que puedan ser accesibles por Podman y los contenedores. Esto se hace para garantizar que los contenedores puedan funcionar correctamente y acceder a los recursos necesarios.

Sin embargo, cuando se trata de otros directorios que no están configurados de forma predeterminada para ser accesibles por Podman y los contenedores, es posible que estos no puedan acceder a ellos debido a las políticas de seguridad de SELinux. SELinux puede aplicar restricciones sobre qué procesos pueden acceder a qué recursos, y si un directorio no está configurado adecuadamente o si está fuera del contexto permitido por SELinux, los contenedores pueden tener dificultades para acceder a él.

Deberemos aplicar etiquetas de contexto adecuadas a los directorios para permitir el acceso de los contenedores según sea necesario. 

## Uso de almacenamiento en contenedores

En la creación de contenedores con `podman run` puedo indicar que vamos a usar almacenamiento para guardar la información de ciertos directorios. Tanto en el caso de uso de volúmenes como en el caso del uso de bind mount podemos indicar el uso de almacenamiento en la creación de un contenedor con las siguientes parámetros del comando `podman run`:

* El parámetro `--volume` o `-v`
* El parámetro `--mount`

En general, `--mount` es más explícito y detallado. La mayor diferencia es que la sintaxis `-v` combina todas las opciones en un solo campo, mientras que la sintaxis `--mount` las separa.

* Si usamos `-v` se debe indicar tres campos separados por dos puntos:
    * El primer campo es el nombre del volumen, debe ser único en una determinada máquina. Para volúmenes anónimos, el primer campo se omite.
    * El segundo campo es la ruta donde se montan el archivo o directorio en el contenedor.
    * El tercer campo es opcional, y es una lista de opciones separadas por comas, la más utilizadas son:
        * `:ro`: Para indicar que el montaje es de sólo lectura.
        * `:z`: Cuando trabajamos en sistemas operativos con SELinux nos permite cambiar la etiqueta del contexto de seguridad del directorio para que sea accesible desde el contenedor. Además el directorio podrá ser compartido con otros contenedores.
        * `:Z`: Cuando trabajamos en sistemas operativos con SELinux nos permite cambiar la etiqueta del contexto de seguridad del directorio para que sea accesible desde el contenedor, pero de forma privada, no se podrá compartir con otros contenedores.
* Si usamos `--mount` hay que indicar un conjunto de datos de la forma `clave=valor`, separados por coma.
    * Clave `type`: Indica el tipo de montaje. Los valores pueden ser `bind`, `volume`, `tmpfs` `devpts`, `glob`, `image` o `ramfs`.
    * Clave `source` o `src`: La fuente del montaje. Se indica el volumen o el directorio que se va montar con bind mount.
    * Clave `dst` o `target`: Será la ruta donde está montado el fichero o directorio en el contenedor. 
    * Se pueden indicar opciones según el tipo de la fuente de montaje, en el caso de los volúmenes y los bind mount, las opciones más utilizadas son:
        * La opción `readonly` o `ro` es optativa, e indica que el montaje es de sólo lectura.
        * `relabel`, puede tener dos valores: `shared` funcionaría de forma similar a cómo lo hace la opción `:z` en el caso de utilizar la sintaxis `-v`, y `private` funcionaría de forma similar a utilizar la opción `:Z`.


## ¿Qué información tenemos que guardar?

¿Qué debemos guardar de forma persistente en un contenedor?

* Los datos de la aplicación.
* Los logs del servicio.
* La configuración del servicio. En este caso podemos añadirla a la imagen, pero será necesaria la creación de una nueva imagen si cambiamos la configuración. Si la guardamos en un volumen hay que tener en cuanta que ese fichero lo tenemos que tener en el entorno de producción (puede ser bueno, porque las configuraciones de los distintos entornos puede variar).

