# Curso Podman 2024

## Contenido

1. Introducción a Podman    
    * [Introducción a los contenedores OCI](contenido/modulo1/contenedores.md)
    * [Aplicaciones para trabajar con contenedores OCI](contenido/modulo1/aplicaciones.md)
    * [Introducción de Podman](contenido/modulo1/podman.md)
    * Instalación de Podman en Linux
    * Instalación de Podman en Windows
2. Ejecución de contenedores OCI con Podman
    * [El "Hola Mundo" en Podman](contenido/modulo2/holamundo.md)
    * [Rootfull us Rootless](contenido/modulo2/introduccion.md)
    * [Ejecución simple de contenedores](contenido/modulo2/contenedor.md)
    * [Ejecutando un contenedor interactivo](contenido/modulo2/interactivo.md)
    * [Ejecutando un contenedor demonio](contenido/modulo2/demonio.md)
    * [Gestión de contenedores](contenido/modulo2/gestion.md)
    * [Limitando los recursos utilizados por un contenedor](contenido/modulo2/recursos.md)
    * [Ejemplo: Ejecución de un contenedor con un servidor web](contenido/modulo2/web.md)
    * [Ejemplo: Ejecución de un contenedor con un servidor de base de datos](contenido/modulo2/mariadb.md)
    * [Introducción a los contenedores rootless](contenido/modulo2/rootless.md)
    * [¿Cómo funcionan los contenedores rootless?](contenido/modulo2/funcionamiento.md)
3. Gestión de imágenes OCI en Podman
    * [Imágenes de contenedor](contenido/modulo3/imagenes.md)
    * [Introducción a los sistemas de archivos de unión](contenido/modulo3/overlay.md)
    * [Almacenamiento de imágenes](contenido/modulo3/almacen_img.md)
    * [Almacenamiento de contenedores](contenido/modulo3/almacen_cont.md)
    * [Almacenamiento de contenedores rootless](contenido/modulo3/rootless.md)
    * [Registros de imágenes](contenido/modulo3/registro.md)
    * [Gestión de imágenes](contenido/modulo3/gestion.md)
    * [Ejemplo: Desplegando la aplicación Drupal](contenido/modulo3/drupal.md)
4. Almacenamiento y redes en Podman
    * [Almacenamiento en Podman](contenido/modulo4/almacenamiento.md)
    * [Trabajando con volúmenes](contenido/modulo4/volumen.md)
    * [Trabajando con bind mount](contenido/modulo4/bindmount.md)
    * [Trabajando con almacenamiento en contenedores rootless](contenido/modulo4/almacenamiento_rootless.md)
    * [Redes en Podman](contenido/modulo4/redes.md)
    * [Redes en contenedores rootful](contenido/modulo4/bridge.md)
    * [Gestión de redes definida por el usuario](contenido/modulo4/usuario.md)
    * [Uso de la red bridge definidas por el usuario](contenido/modulo4/usuario2.md)
    * [Redes en contenedores rootless](contenido/modulo4/red_rootless.md)
    * Ejemplo 1: Despliegue de la aplicación Guestbook
    * Ejemplo 2: Despliegue de la aplicación Temperaturas
    * Ejemplo 3: Despliegue de WordPress + MariaDB
    * Ejemplo 4: Despliegue de Apache Tomcat + nginx

5. Ejecución avanzada de contenedores con Podman
    * Trabajando con Pods en Podman
    * Generación de un archivo YAML de Kubernetes con Podman
    * Gestionando contenedores y Pods con systemd
    * Especificación Compose
    * Creando escenarios multicontenedor con Compose
    * El fichero compose.yaml
    * El comando podman-compose
    * Almacenamiento con Compose
    * Redes con Compose
    * Ejemplo 1: Despliegue de la aplicación Guestbook
    * Ejemplo 2: Despliegue de la aplicación Temperaturas
    * Ejemplo 3: Despliegue de WordPress + MariaDB
    * Ejemplo 4: Despliegue de Apache Tomcat + nginx
    * Uso de parámetros con Compose
    * Eliminar objetos Podman no utilizados

6. Creación de imágenes con Podman
    * Introducción a la construcción y distribución de imágenes OCI
    * El fichero Containerfile 
    * Creación de imágenes con podman build
    * Distribución de imágenes
    * Ejemplo 1: Construcción de imágenes con una página estática
    * Ejemplo 2: Construcción de imágenes con una una aplicación PHP
    * Ejemplo 3: Construcción de imágenes con una una aplicación Python
    * Ejemplo 4: Construcción de imágenes configurables con variables de entorno
    * Ejemplo 5: Configuración de imágenes con una aplicación Java
    * Creación de imágenes con Compose
    * Uso de ficheros Containerfile parametrizados
    
7. Gestión de imágenes OCI con Buildah y Skopeo
    * Introducción a Buildah y Skopeo
    * Crear una imagen con Buildah desde cero
    * Crear una imagen con Buildah desde un contenedor
    * Crear una imagen con Buildah desde un Containerfile
    * Introducción a Skopeo
    * Copiar imágenes OCI entre distintos registros
    * Inspeccionar una imagen OCI remota 
    * Gestión de imágenes OCI en registros remotos
    * Firmar imágenes OCI 

8. Podman Desktop