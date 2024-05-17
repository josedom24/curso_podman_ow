# Trabajando con Pods en Podman

## ¿Qué es un Pod?

* Un Pod es un concepto que proviene de Kubernetes.
* En Kubernetes los contenedores se ejecutan en Pods. En inglés Pod significa "vaina", y podemos entender un Pod como una envoltura que contiene uno o varios contenedores.
* Un Pod representa un conjunto de contenedores que comparten almacenamiento y una única IP.
* Un Pod recibe una dirección IP que es compartida por todos los contenedores.
* Esto significa que todos los servicios que se ejecutan en los diferentes contenedores pueden referirse entre sí como localhost, mientras que los contenedores externos seguirán contactando con la dirección IP del Pod. 

## Gestión de Pods con Podman

![pod](img/podman-pod-architecture.png)

* Cada Pod en Podman incluye un contenedor llamado **"infra"**.   
    * Este contenedor no hace nada más que dormir. 
    * Su propósito es mantener los espacios de nombres asociados con el pod y permitir a Podman conectar otros contenedores al pod. 
    * Esto le permite iniciar y detener contenedores dentro del Pod.
    * El contenedor infra por defecto está basado en la imagen `localhost/podman-pause`.
* Como vemos en la imagen el Pod puede estar formada por varios contenedores:
    * Si seguimos la filosofía de Kubernetes cada Pod tendrá un contenedor principal encargado de ofrecer el servicio.
    * Si necesitamos realizar procesos auxiliares fuertemente acoplados con el principal, podemos tener contenedores secundarios que llamamos contenedores sidecar. Por ejemplo: Un servidor web nginx con un servidor de aplicaciones PHP-FPM, que se implementaría mediante un solo Pod, pero ejecutando un proceso de nginx en un contenedor y otro proceso de php-fpm en otro contenedor.
    * Evidentemente también podemos utilizar Pods multicontenedores para desplegar aplicaciones que necesitan más de un servicio para su funcionamiento.
* En el diagrama anterior, también observamos el proceso **conmon**, este es el monitor del contenedor.  Es un pequeño programa en C cuyo trabajo es monitorizar los contenedores y permitimos conectarnos a ellos. Cada contenedor tiene su propia instancia de conmon, independientemente se ejecute dentro de un Pod:
    ```
    $ podman run -d quay.io/libpod/banner
    
    $ pstree
    systemd─┬─...
            ├─conmon───nginx───2*[nginx]
    ```
* En Podman podemos trabajar con Pods en entornos rootful y rootless.
