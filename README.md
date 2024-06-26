# Curso Podman 2024

## Ejemplos

* [Repositorio con el código de los ejemplos](https://github.com/josedom24/ejemplos_curso_podman_ow)

## Contenido

1. Introducción a Podman    
    * [Introducción a los contenedores OCI](contenido/modulo1/contenedores.md) (P)
    * [Aplicaciones para trabajar con contenedores OCI](contenido/modulo1/aplicaciones.md) (P)
    * [Introducción de Podman](contenido/modulo1/podman.md) (P)
    * [Instalación de Podman en Linux](contenido/modulo1/linux.md)
    * [Instalación de Podman en Windows/macOS](contenido/modulo1/windows.md)
2. Ejecución de contenedores OCI con Podman
    * [El "Hola Mundo" en Podman](contenido/modulo2/holamundo.md)
    * [Ejecución simple de contenedores](contenido/modulo2/contenedor.md)
    * [Ejecución de contenedores interactivos](contenido/modulo2/interactivo.md)
    * [Ejecución de contenedores demonios](contenido/modulo2/demonio.md)
    * [Ejecución de un contenedor demonio con un servidor web](contenido/modulo2/web.md)
    * [Obteniendo información de los contenedores](contenido/modulo2/informacion.md)
    * [Configuración de contenedores](contenido/modulo2/configuracion.md)    
    * [Ejecución de un contenedor con un servidor de base de datos](contenido/modulo2/mariadb.md)    
    * [Modos de funcionamiento de los contenedores](contenido/modulo2/funcionamiento.md)
    * [¿Cómo funcionan los contenedores rootless?](contenido/modulo2/rootless.md) (P)
3. Gestión de imágenes OCI en Podman
    * [Introducción a las imágenes OCI](contenido/modulo3/imagenes.md) (P)
    * [Introducción al formato de imagen OCI](contenido/modulo3/formato.md) (P)
    * [Almacenamiento de imágenes](contenido/modulo3/almacen_img.md) (P)
    * [Ahorro de espacio de almacenamiento](contenido/modulo3/ahorro_almacenamiento.md)
    * [Almacenamiento de contenedores](contenido/modulo3/almacen_cont.md) (P)
    * [Almacenamiento de contenedores rootless](contenido/modulo3/rootless.md) (P)
    * [Gestión de imágenes](contenido/modulo3/gestion.md)
    * [Obteniendo información de las imágenes](contenido/modulo3/informacion.md)
    * [Ejemplo: Desplegando la aplicación Drupal](contenido/modulo3/drupal.md)
4. Almacenamiento y redes en Podman
    * [Almacenamiento en Podman](contenido/modulo4/almacenamiento.md) (P)
    * [Trabajando con volúmenes](contenido/modulo4/volumen.md)
    * [Trabajando con bind mount](contenido/modulo4/bindmount.md)
    * [Trabajando con almacenamiento en contenedores rootless](contenido/modulo4/almacenamiento_rootless.md)
    * [Redes en Podman](contenido/modulo4/redes.md) (P)
    * [Uso de la red bridge por defecto](contenido/modulo4/bridge.md)
    * [Gestión de redes definidas por el usuario](contenido/modulo4/usuario.md)
    * [Uso de la red bridge definidas por el usuario](contenido/modulo4/usuario2.md)
    * [Uso de la red host en Podman](contenido/modulo4/host.md)
    * [Redes en contenedores rootless](contenido/modulo4/red_rootless.md)
    * [Despliegue de la aplicación Guestbook](contenido/modulo4/guestbook.md)
    * [Despliegue de la aplicación Temperaturas](contenido/modulo4/temperaturas.md)

5. Gestión de Pods en Podman
    * [Trabajando con Pods en Podman](contenido/modulo5/pod.md) (P)
    * [Gestión de Pods](contenido/modulo5/gestion.md)
    * [Funcionamiento de la red en un Pod](contenido/modulo5/red.md)
    * [Almacenamiento compartido entre los contenedores de un Pod](contenido/modulo5/almacenamiento.md)
    * [Ejemplo: Despliegue de WordPress + MariaDB en un Pod](contenido/modulo5/wordpress.md)
    * [Ejemplo: Despliegue de WordPress + MariaDB en un escenario multipod](contenido/modulo5/wordpress2.md)
    * [Ejemplo: nginx + fpm-php](contenido/modulo5/nginx.md)
    * [Generación de un archivo YAML de Kubernetes con Podman](contenido/modulo5/kubernetes.md)
    * [Ejecutando recursos de Kubernetes en Podman](contenido/modulo5/kubernetes2.md)

6. Gestionando recursos de Podman con Systemd y Quadlet
    * [Introducción a Quadlet](contenido/modulo6/quadlet.md) (P)
    * [Ejecución de contenedores con Systemd y Quadlet](contenido/modulo6/contenedor.md)
    * [Gestionando volúmenes y redes con Systemd y Quadlet](contenido/modulo6/vol_redes.md)
    * [Ejecución de Pods con Systemd y Quadlet](contenido/modulo6/pod.md)
    * [Ejemplo: Despliegue de WordPress + MariaDB con Systemd y Quadlet](contenido/modulo6/wordpress.md)

7. Gestionando escenarios multicontenedor con podman-compose
    * [Creando escenarios multicontenedor con Compose](contenido/modulo7/compose.md) (P)
    * [El fichero compose.yaml](contenido/modulo7/compose_yaml.md)
    * [El comando podman-compose](contenido/modulo7/podman_compose.md)
    * [Almacenamiento y redes con Compose](contenido/modulo7/almacenamiento_redes.md)
    * [Uso de parámetros con Compose](contenido/modulo7/variables.md)
    * [Creación de contenedores rootless conectados a la red slirp4netns](contenido/modulo7/rootless.md)
    * [Despliegue de la aplicación Guestbook](contenido/modulo7/guestbook.md)
    * [Despliegue de la aplicación Temperaturas](contenido/modulo7/temperaturas.md)
    
8. Gestión de imágenes OCI con Podman
    * [Construcción y distribución de imágenes OCI](contenido/modulo8/introduccion.md) (P)
    * [Construcción de imágenes a partir de un contenedor](contenido/modulo8/contenedor.md)
    * [El fichero Containerfile](contenido/modulo8/containerfile.md) (P)
    * [Construcción de imágenes con podman build](contenido/modulo8/build.md)
    * [Construcción de imágenes configurables con variables de entorno](contenido/modulo8/configuracion.md)
    * [Construcción de imágenes con Compose](contenido/modulo8/compose.md)
    * [Distribución de imágenes OCI](contenido/modulo8/distribucion.md)
    * [Uso de ficheros Containerfile parametrizados](contenido/modulo8/variables.md)
    
9. Gestión de imágenes OCI con Buildah y Skopeo
    * [Introducción a Buildah y Skopeo](contenido/modulo9/introduccion.md) (P)
    * [Construcción de imágenes desde una imagen base con Buildah](contenido/modulo9/base.md)
    * [Construcción de imágenes desde cero con Buildah](contenido/modulo9/scratch.md)
    * [Construcción de imágenes desde un Containerfile con Buildah](contenido/modulo9/build.md)
    * [Distribución de imágenes con Buildah](contenido/modulo9/distribucion.md)
    * [Gestión de imágenes con Skopeo](contenido/modulo9/skopeo.md)
    
10. Podman Desktop
    * [Introducción a Podman Desktop](contenido/modulo10/introduccion.md)
    * [Gestión de imágenes en Podman Desktop](contenido/modulo10/imagenes.md)
    * [Gestión de contenedores en Podman Desktop](contenido/modulo10/contenedor.md)
    * [Gestión de Pods en Podman Desktop](contenido/modulo10/pod.md)
    * [Gestión de volúmenes en Podman Desktop](contenido/modulo10/volumenes.md)
    * [Construcción de imágenes en Podman Desktop](contenido/modulo10/build.md)
    
