# Introducción a los contenedores OCI

Un **contenedor** es un conjunto de procesos aislados, que se ejecutan en un servidor, y que acceden a un sistema de ficheros propio, tienen una configuración de red propia y acceden a los recursos del host (memoria y CPU). Utilizan para su funcionamiento el núcleo del host donde se ejecutan.

El uso de contenedores nos proporciona un **sistema de virtualización**, ya que nos permiten imitar las características del hardware y crear un sistema informático virtual. 

En concreto, al usar contenedores estamos usando **virtualización ligera**, o  **virtualización a nivel de sistema operativo**, o **virtualización basada en contenedores**.

## Clasificación de contenedores

Podemos hacer la siguiente clasificación de contenedores según el uso que hacemos de ellos:

* **Contenedores de Sistemas**: El uso que se hace de ellos es muy similar al que hacemos sobre una máquina virtual: se accede a ellos (normalmente por ssh), se instalan servicios, se actualizan, ejecutan un conjunto de procesos, ... Ejemplo: LXC(Linux Container).
* **Contenedores de Aplicación**: Se suelen usar para el despliegue de aplicaciones web. Ejemplo: Docker, Podman, ...

## Contenedores y aplicaciones

¿Qué aplicaciones web son más idóneas para desplegar en contenedores?

* Si tenemos aplicaciones monolíticas, vamos a usar un esquema multicapa, es decir cada servicio (servicio web, servicio de base de datos, ... ) se va a desplegar en un contenedor.
* Realmente, las aplicaciones que mejor se ajustan al despliegue en contenedores son las desarrolladas con microservicios:
    * Cada componente de la aplicación (“microservicio”) se puede desplegar en un contenedor.
    * Comunicación vía HTTP API REST y colas de mensajes.
    * Facilita enormemente las actualizaciones de versiones de cada componente.
    * ...

## Ventajas del uso de contenedores de aplicaciones

Los contenedores simplifican la producción, distribución, descubrimiento y uso de aplicaciones con todas sus dependencias y archivos de configuración empaquetados. Por lo tanto las ventajas de su uso son: 

1. **Portabilidad**: Los contenedores encapsulan una aplicación y todas sus dependencias de manera aislada. Esto facilita la portabilidad, ya que los contenedores pueden ejecutarse de manera consistente en diferentes entornos, como desarrollo, pruebas y producción.
2. **Aislamiento**: Los contenedores proporcionan un nivel de aislamiento entre la aplicación y el sistema operativo del anfitrión. Esto asegura que las aplicaciones se ejecuten sin interferencias con otras aplicaciones o componentes del sistema, evitando conflictos de dependencias y problemas de compatibilidad.
3. **Eficiencia en el uso de recursos**: Al compartir el núcleo del sistema operativo y solo incluir las bibliotecas y dependencias necesarias, los contenedores son más eficientes en términos de recursos en comparación con las máquinas virtuales. Esto permite una utilización más efectiva de los recursos del sistema y una mayor densidad de aplicaciones por servidor.
4. **Despliegue rápido**: Los contenedores permiten la creación, el despliegue y la escalabilidad rápida de aplicaciones. La capacidad de implementar contenedores en cuestión de segundos o minutos facilita la entrega continua y el despliegue ágil de aplicaciones.

## Open Container Initiative

La [**Open Container Initiative (OCI)**](https://opencontainers.org/), es un proyecto de la **Linux Foundation** para diseñar un estándar abierto para **virtualización basada en contenedores**. El objetivo con estos estándares es asegurar que las plataformas de contenedores no estén vinculadas a ninguna empresa o proyecto concreto.

Se funda en 2015, y actualmente son muchas las empresas que forman parte de esta iniciativa, y cuyo objetivo es desarrollar las siguientes especificaciones:​

* Una especificación de **entorno de ejecución de contenedores** (Open Container Initiative Runtime Specification normalmente abreviado en OCI Runtime Specification). Describe cómo debe proceder un OCI Runtime para crear un contenedor a partir de una imagen. Los OCI runtime más utilizados son **runC** y **crun**.
* Una especificación de **formato de imagen** (Open Container Initiative Image Format normalmente abreviado en OCI Image Format). Determina el formato para empaquetar la imagen del contenedor de software.
* Una especificación de **distribución de imágenes**(Open Container Initiative Distribution Specification normalmente abreviado en OCI Distribution Specification). Su objetivo es estandarizar la distribución de imágenes de contenedores.

