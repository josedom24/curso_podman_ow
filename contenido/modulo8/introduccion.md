# Construcción y distribución de imágenes OCI

## Construcción de imágenes OCI

Hasta ahora hemos creado contenedores a partir de las imágenes que encontramos en distintos registros de imágenes OCI. Estas imágenes las han creado otras personas.

Para crear un contenedor que sirva nuestra aplicación, tendremos que crear una imagen personalizada, es lo que llamamos **"dockerizar"** una aplicación.

Tenemos dos herramientas para realizar la construcción de una imagen OCI:

* **Podman**:Podman nos ofrece distintas instrucciones para construir imágenes OCI. Con Podman podemos crear nuevas imágenes usando dos mecanismos principales:
    * **A partir de un contenedor**, podemos crea una nueva imagen usando el comando `podman commit`.
    * **Automatizar la construcción** de una imagen OCI declarando los comandos que hay que ejecutar en un fichero llamado `Containerfile` y usando el comando `podman build` para realizar la construcción. Este método nos ofrece algunas ventajas respecto al primero:
        * **Podremos reproducir la imagen fácilmente** ya que en el fichero `Containerfile` tenemos todas y cada una de las órdenes necesarias para la construcción de la imagen. Además el fichero `Containerfile` se puede distribuir de manera muy sencilla y versionar usando un sistema de control de versiones.
        * De manera sencilla podemos **cambiar la imagen base** usando un fichero `Containerfile`, únicamente tendremos que modificar la primera línea de ese fichero como explicaremos posteriormente.
* **Buildah**: Es una herramienta específica para la **construcción de imágenes de contenedores** sin necesidad de ejecutar un demonio. Permite a los usuarios construir imágenes OCI usando varios mecanismos:
    * **A partir de una imagen base**.
    * **A parir de un fichero `Containerfile`**.
    * **Desde 0**: Instalando los paquetes que necesitemos de la distribución deseada.

## Distribución de imágenes OCI

Una vez que hemos creado nuestra imagen personalizada, es hora de distribuirla para desplegarla en el entorno de producción. 

Hay que recordar que tenemos varios medios de almacenamiento de imágenes, que llamamos **transportes de imágenes** y nos permiten almacenar una imagen. Los **transportes de imágenes OCI** son los siguientes:

* **docker**: Este es el transporte por defecto. Hace referencia a una imagen almacenada en un **Registro remoto de imágenes**. Los registros almacenan y comparten imágenes (por ejemplo, `docker.io` y `quay.io`).
* **containers-storage**: Hace referencia a una imagen guarda en un registro local de Podman.
* **oci**: Hace referencia a una imagen con formato OCI, su configuración y capas e encuentran en el directorio local como archivos individuales.
* **oci-archive**: Hace referencia a una imagen con formato OCI comprimida en una archivo tar.

Por lo tanto podremos usar cualquier **transporte de imagen** para enviar y distribuir una imagen OCI. Para realizar la distribución tenemos dos herramientas:

* **Podman**: Podman nos ofrece varias instrucciones para usar los **transportes de imágenes OCI** para distribuir una imagen OCI:
    * Utilizando los comandos `podman save / podman load` podemos guardar y recuperar una imagen que tenemos en el registro local en un fichero comprimido (usando el medio de transporte `oci-archive`) o en un directorio con ele contenido de la imagen (usando el medio de transporte `oci`).
    * Utilizando los comando `podman push / podman pull` que aunque también permite trabajar con imágenes guards en ficheros comprimidos o directorios, se suelen usar almacenar y recuperar imágenes en registros de imágenes remotos y locales.
* **Skopeo**: Skopeo es una herramienta específica para la gestión de imágenes OCI, que entre otras cosas nos permite copiar imágenes usando los distintos medios de transportes de imágenes.