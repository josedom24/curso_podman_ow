# Rootful us Rootless

Los contenedores Podman tienen dos modos de ejecución:

* **Contenedor rootful**: es un contenedor ejecutado por `root` en el host. Por lo tanto, tiene acceso a toda la funcionalidad que el usuario `root` tiene. 
    * De todas maneras, es posible que algunos procesos ejecutados en el contenedor no se ejecuten como `root`. 
    * Este modo de funcionamiento puede tener problemas de seguridad, ya que si hay una vulnerabilidad en la funcionalidad, el usuario del contenedor será `root` con los posibles riesgos de seguridad que esto conlleva.
* **Contenedor rootless**: es un contenedor que puede ejecutarse sin privilegios de `root` en el host. 
    * Podman no utiliza ningún demonio y no necesita `root` para ejecutar contenedores.
    * Esto no significa que el usuario dentro del contenedor no sea `root`, aunque sea el usuario por defecto.
    * Si tenemos una vulnerabilidad en la ejecución del contenedor, el atacante no obtendrá privilegios de `root` en el host.
    * Estos contenedores tienen algunas limitaciones:
        * No tienen acceso a todas las características del sistema operativo.
        * No se pueden crear contenedores que se unan a puertos privilegiados (menores que 1024).
        * Algunos modos de almacenamiento pueden dar problemas.
        * Por defecto, no se puede hacer `ping` a servidores remotos.
        * No pueden gestionar las redes del host.
        * [Más limitaciones](https://github.com/containers/podman/blob/master/rootless.md)

Los primeros ejemplos que vamos a ver en este módulo lo vamos a ejecutar sobre contenedores Rootful, pero serían válidos también para contenedores rootless. Cuando vayamos a ejecutar un contenedor Rootful voy a usar el comando `sudo` para recordar que lo estamos ejecutando con el usuario `root`.