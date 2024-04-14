# Curso Podman 2024

## Ejemplos

* [Repositorio con el código de los ejemplos](https://github.com/josedom24/ejemplos_curso_podman_ow)

## Contenido

1. Introducción a Podman    
    * [Introducción a los contenedores OCI](contenido/modulo1/contenedores.md)
    * [Aplicaciones para trabajar con contenedores OCI](contenido/modulo1/aplicaciones.md)
    * [Introducción de Podman](contenido/modulo1/podman.md)
    * Instalación de Podman en Linux
    * Instalación de Podman en Windows
2. Ejecución de contenedores OCI con Podman
    * [El "Hola Mundo" en Podman](contenido/modulo2/holamundo.md)
    * [Ejecución simple de contenedores](contenido/modulo2/contenedor.md)
    * [Ejecución de contenedores interactivos](contenido/modulo2/interactivo.md)
    * [Ejecución de contenedores demonios](contenido/modulo2/demonio.md)
    * [Gestión de contenedores](contenido/modulo2/gestion.md)
    * [Configuración de contenedores](contenido/modulo2/configuracion.md)    
    * [Modos de funcionamiento de los contenedores](contenido/modulo2/funcionamiento.md)
    * [¿Cómo funcionan los contenedores rootless?](contenido/modulo2/rootless.md)
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

5. Gestión de Pods en Podman
    * [Trabajando con Pods en Podman](contenido/modulo5/pod.md)
    * [Gestión de Pods](contenido/modulo5/gestion.md)
    * [Funcionamiento de la red en un Pod](contenido/modulo5/red.md)
    * [Almacenamiento compartido entre los contenedores de un Pod](contenido/modulo5/almacenamiento.md)
    * [Ejemplo: Despliegue de WordPress + MariaDB en un Pod](contenido/modulo5/wordpress.md)
    * [Ejemplo: Despliegue de WordPress + MariaDB en un escenario multipod](contenido/modulo5/wordpress2.md)
    * [Creación de Pods en modo rootless](contenido/modulo5/rootless.md)
    * [Generación de un archivo YAML de Kubernetes con Podman](contenido/modulo5/kubernetes.md)
    * Ejecutando recursos de Kubernetes en Podman

6. Gestionando recursos de Podman con Systemd y Quadlet
    * [Introducción a Quadlet](contenido/modulo6/quadlet.md)
    * [Ejecución de contenedores con Systemd y Quadlet](contenido/modulo6/contenedor.md)
    * [Gestionando volúmenes y redes con Systemd y Quadlet](contenido/modulo6/vol_redes.md)
    * [Ejecución de Pods con Systemd y Quadlet](contenido/modulo6/pod.md)
    * Ejemplo: Despliegue de WordPress + MariaDB con systemd y Quadlet
    * Ejecución de contenedores rootless con Systemd y Quadlet

7. Gestionando escenarios multicontenedor con podman-compose
    * [Creando escenarios multicontenedor con Compose](contenido/modulo7/compose.md)
    * [El fichero compose.yaml](contenido/modulo7/compose_yaml.md)
    * [El comando podman-compose](contenido/modulo7/podman_compose.md)
    * [Almacenamiento y redes con Compose](contenido/modulo7/almacenamiento_redes.md)
    * Ejemplo 1: Despliegue de la aplicación Guestbook
    * Ejemplo 2: Despliegue de la aplicación Temperaturas
    * Ejemplo 3: Despliegue de WordPress + MariaDB
    * Ejemplo 4: Despliegue de Apache Tomcat + nginx
    * [Uso de parámetros con Compose](contenido/modulo7/variables.md)
    * Creación de contenedores rootless con podman-compose
    * Creación de Pods con podman-compose
    * Eliminar objetos Podman no utilizados


8. Gestión de imágenes OCI con Podman
    * [Construcción y distribución de imágenes OCI](contenido/modulo8/introduccion.md)
    * El fichero Containerfile 
    * Construcción de imágenes con podman build
    * Distribución de imágenes
    * Ejemplo 1: Construcción de imágenes con una página estática
    * Ejemplo 2: Construcción de imágenes con una una aplicación PHP
    * Ejemplo 3: Construcción de imágenes con una una aplicación Python
    * Ejemplo 4: Construcción de imágenes configurables con variables de entorno
    * Ejemplo 5: Configuración de imágenes con una aplicación Java
    * Construcción de imágenes con Compose
    * Uso de ficheros Containerfile parametrizados
    
9. Gestión de imágenes OCI con Buildah y Skopeo
    * Introducción a Buildah
    * Crear una imagen con Buildah desde una imagen base
    * Crear una imagen con Buildah desde un Containerfile
    * Crear una imagen con Buildah desde cero
    * Introducción a Skopeo
    * Copiar imágenes OCI entre distintos registros
    * Inspeccionar una imagen OCI remota 
    * Gestión de imágenes OCI en registros remotos

10. Podman Desktop