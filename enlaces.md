* Using volumes with rootless podman, explained (https://www.tutorialworks.com/podman-rootless-volumes/)
* A Practical Introduction to Container Terminology (https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction#)
* Contenedores Red Hat 
    * https://access.redhat.com/documentation/es-es/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/index
    * https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/building_running_and_managing_containers/index
* What's the recommended way of installing Podman 4 in Ubuntu 22.04? (https://askubuntu.com/questions/1414446/whats-the-recommended-way-of-installing-podman-4-in-ubuntu-22-04)
* Imágenes OCI (https://ravichaganti.com/blog/2022-10-28-understanding-container-images-oci-image-specification/)

## rootless
* https://kcore.org/2023/12/13/adventures-with-rootless-containers/
* https://www.redhat.com/sysadmin/container-networking-podman
* [Containers: Rootful, Rootless, Privileged and Super Privileged](https://infosecadalid.com/2021/08/30/containers-rootful-rootless-privileged-and-super-privileged/)
## Firmar imágenes

* https://gist.github.com/saschagrunert/1d0b3cc40ce0418558d7c2a5a8fe8a1c

## volumen rootless

* https://blog.christophersmart.com/2021/01/31/volumes-and-rootless-podman/
* https://blog.christophersmart.com/2021/01/31/podman-volumes-and-selinux/
* Más artículos sobre podman:https://blog.christophersmart.com/?s=podman

## rootless

* https://www.redhat.com/sysadmin/rootless-containers-podman
    * https://www.redhat.com/en/blog/understanding-root-inside-and-outside-container
* https://www.redhat.com/sysadmin/behind-scenes-podman
* https://www.redhat.com/en/blog/understanding-root-inside-and-outside-container
* https://www.redhat.com/sysadmin/user-flag-rootless-containers

La instrucción systemctl --user start dbus en Linux se utiliza para iniciar el servicio del sistema de mensajería de la base de datos de D-Bus (DBus) en el contexto del usuario actual.

Aquí hay una explicación de cada parte de la instrucción:

    systemctl: Es una utilidad de control del sistema que se utiliza para administrar servicios de systemd. systemctl puede usarse para iniciar, detener, reiniciar y administrar otros aspectos de los servicios del sistema.

    --user: Esta opción indica que el comando systemctl debe operar en el contexto del usuario actual en lugar de en el contexto del sistema en general. Esto significa que las acciones tomadas con --user afectarán solo al usuario actual y no requerirán permisos de superusuario (root).

    start: Esta subcomanda de systemctl indica que se debe iniciar un servicio. En este caso, se está iniciando el servicio DBus.

    dbus: Es el nombre del servicio que se desea iniciar. D-Bus es un sistema de comunicación entre procesos que permite que los procesos se comuniquen entre sí y proporciona un bus de mensajes para aplicaciones.

En resumen, la instrucción systemctl --user start dbus inicia el servicio de D-Bus en el contexto del usuario actual sin necesidad de privilegios de superusuario. Esto asegura que los procesos en el entorno de usuario puedan comunicarse a través del sistema de mensajería de D-Bus.



quay.io/libpod/banner 

Imagen pequeña con servidor web


# Podman Desktop

* https://linuxiac.com/podman-desktop-1-8-integrates-kubernetes-explorer-and-enhanced-ui/