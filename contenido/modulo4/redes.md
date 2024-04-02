# Redes en Podman

Dependiendo del tipo de contenedor que creemos, Podman utilizará distintos mecanismos para ofrecer conectividad al contenedor:

* **Contenedores rootful**: Podman 4.0 utiliza un driver de red llamado [netavark](https://github.com/containers/netavark) para ofrecer a los contenedores una dirección IP enrutable. El driver netavark sigue las especificaciones establecidas por el proyecto [CNI](https://www.cni.dev/) (Container Network Interface) que estandariza los medios de comunicación de red que utilizan los contenedores OCI.
* **Contenedores rootless**: Cuando usamos contenedores rootless, tenemos una limitación debida a que los usuarios sin privilegios no pueden crear interfaces de red en el host. En este caso se utiliza, por defecto el proyecto [slirp4netns](https://github.com/rootless-containers/slirp4netns) que nos proporciona conectividad a los contenedores pero sin ofrecerles una dirección IP enrutable. En la nueva versión de Podman, la 5.0 este proyecto se ha sustituido por otro proyecto con las mismas características llamado [pasta](https://passt.top/passt/about/).

