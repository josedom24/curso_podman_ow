# Curso Podman 2024

## Contenido

1. Introducción a Podman    
    * Introducción a los contenedores
    * Introducción a Podman
    * Instalación de Podman en Linux
    * Instalación de Podman en Windows
    * El "Hola Mundo" en Podman
2. Gestión de imágenes con Podman
    * Imágenes OCI
    * OverlayFS: Composición de imágenes OCI
    * Registros de imágenes OCI
    * Gestión de imágenes OCI
    * Almacenamiento de imágenes OCI
    * Ejemplo: Desplegando la aplicación MediaWiki
3. Ejecución de contenedores con Podman
    * Rootfull us Rootless
    * Ejecución simple de contenedores
    * Ejecutando un contenedor interactivo
    * Ejecutando un contenedor demonio
    * Gestión de contenedores
    * Ejemplo: Ejecución de un contenedor con un servidor web
    * Ejemplo: Ejecución de un contenedor con un servidor de base de datos
4. Almacenamiento y redes en Podman
    * Almacenamiento en Podman
    * Trabajando con volúmenes en Podman
    * Trabajando con bind mount en Podman
    * Redes en Podman
    * Redes en contenedores rootful
    * Gestión de redes definida por el usuario
    * Redes en contenedores rootless
    * Ejemplo 1: Contenedor NextCloud con almacenamiento persistente
    * Ejemplo 2: Contenedor MariaDB con almacenamiento persistente    

5. Ejecución avanzada de contenedores con Podman
    * Trabajando con Pods con Podman
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