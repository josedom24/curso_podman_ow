# Introducción a Podman

## Podman

* **Podman (the POD MANager)** es una herramienta para gestionar contenedores e imágenes OCI, volúmenes montados en esos contenedores y Pods (grupos de contenedores). 
* Podman ejecuta contenedores en **Linux**, pero también puede utilizarse en sistemas **Mac y Windows** utilizando una máquina virtual gestionada por Podman. 
* Podman se basa en **libpod**, una biblioteca para la gestión del ciclo de vida de los contenedores. La librería libpod proporciona APIs para la gestión de contenedores, Pods, imágenes de contenedores y volúmenes.
* Podman es una herramienta nativa de Linux, de **código abierto** y **sin demonio**, diseñada para facilitar la búsqueda, ejecución, creación, uso compartido y despliegue de aplicaciones mediante contenedores e imágenes OCI.
* Podman proporciona una interfaz de línea de comandos (CLI) familiar para cualquiera que haya utilizado el motor de contenedores Docker. 
* Podman se basa en un OCI runtime (runc, crun, ...) para interactuar con el sistema operativo y crear los contenedores en ejecución. 



## Características principales de Podman

### Línea de comandos amigable

Podman se ha desarrollado después de Docker, por lo que sus creadores se han basado en muchas de sus funcionalidades. Entre ellas, el CLI, las opciones de línea de comandos de Podman son muy similares al CLI de Docker, por lo que en un primer momento basta con cambiar el comando `docker` por `podman` y podemos empezar a trabajar con contenedores. Muchas distribuciones Linux ofrecen un paquete llamado `podman-docker` que añade un alias para que al poner el comando `docker` se ejecute el comando `podman`.


### Contenedores rootless

Probablemente la característica más significativa de Podman es su capacidad para ejecutar contenedores por un usuario sin privilegios. No es necesario usar `root` para la ejecución de contenedores.

Esto es útil, cuando queremos que cualquier usuario del sistema pueda ejecutar contenedores y construir imágenes de contenedores, sin requerir acceso de `root`. 

Esta característica nos ofrece un alto grado de seguridad en la ejecución de contenedores.

Aunque en Docker también se puede hacer uso de esta [característica](https://docs.docker.com/engine/security/rootless/), su implantación se ha introducido en versiones más nuevas del producto. Sin embargo, en Podman está característica fue desarrolla desde el comienzo del proyecto.

### Arquitectura Fork/Exec

Docker posee una arquitectura cliente-servidor. El cliente Docker se conecta al demonio Docker para gestionar el contenedor. Podemos entender el demonio Docker como un intermediario entre el cliente Docker y el OCI runtime que gestiona el contenedor. Veamos el diagrama:

![docker](img/docker.png)

En resumen, el cliente Docker se comunica con el demonio Docker, que a su vez se comunica con el demonio containerd, que finalmente lanza un OCI runtime como runc para lanzar el contenedor. La gestión de contenedores es compleja, y cualquier fallo en los elementos involucrados pueden hacer que el contenedor no funcione de manera adecuada.

Podman sigue el modelo Fork/Exec, que tradicionalmente ha funcionado en los sistemas Unix: cuando ejecutamos un nuevo programa, un proceso padre (por ejemplo, `bash`) ejecuta el nuevo programa como un proceso hijo.

En el caso de Podman, al crear un contenedor se crea un proceso hijo, del proceso correspondiente al OCI runtime. Veamos un esquema de este mecanismo:

![podman](img/podman.png)

Este mecanismo de creación de contenedores es mucho más sencillo y nos proporciona una manera más segura de trabajar con contenedores.

### Podman no tiene demonio

Como hemos visto anteriormente, Podman es un software *daemonless*. La diferencia fundamental entre Podman y Docker, es que Podman no tiene un demonio en ejecución. Podman realiza las mismas operaciones que Docker sin necesidad de tener un proceso demonio en ejecución que gestione el ciclo de vida de los contenedores.

Sin embargo para permitir que otros programas puedan usar Podman como gestor de contenedores, Podman puede ofrecer una API REST compatible con la ofrecida por Docker. 

### Integración con systemd

Systemd es el principal sistema de inicialización en los sistemas operativos Linux. Es el responsable de crear el proceso `init` que inicializa el espacio de usuario durante el proceso de arranque de Linux y gestionar posteriormente todos los demás procesos.

Podman puede integrar completamente la ejecución de contenedores con el sistema de systemd. Por lo tanto, podemos usar systemd para gestionar el ciclo de vida de los contenedores.

Los contenedores trabajan como unidades de systemd que podemos invocar como si fueran cualquier otro proceso.

### Pods

Una de las ventajas de Podman se el trabajo con Pods. Un Pod es una envoltura ("vaina") de uno o más contenedores, con recursos compartidos de almacenamiento y red, y una especificación de cómo ejecutar los contenedores.

Podman puede trabajar con un solo contenedor a la vez, o puede gestionar grupos de contenedores juntos en un Pod. Los Pods permiten agrupar varios servicios para formar un servicio mayor gestionado como una entidad única. 

Además, Podman es capaz de generar archivos YAML de de Kubernetes a partir de contenedores y Pods en ejecución.

### Registros de contenedores personalizables

Con Podman podemos gestionar imágenes OCI usando su nombre corto, sin necesidad de indicar el nombre del registro. 

Por ejemplo, el nombre de imagen `ubi8`corresponde al nombre completo `registry.access.redhat.com/library/ubi8:latest`, donde se indica los nombres del registro, del repositorio, de la imagen y de la etiqueta.

En Podman podemos especificar múltiples registros, de tal forma que cuando indicamos el nombre de una imagen, se nos da a elegir entre los distintos registros que tenemos configurados.

