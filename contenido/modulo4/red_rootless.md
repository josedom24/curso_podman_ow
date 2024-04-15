# Redes en contenedores rootless

Cuando trabajamos con contenedores rootless, tenemos una limitación ya que los usuarios sin privilegios no pueden crear interfaces de red.

Por lo tanto tenemos que usar un mecanismo que se ejecute en el espacio de usuario que nos permite ofrecer conectividad a los contenedores rootless. 

En nuestro caso vamos a usar el proyecto **slirp4netns** que crea un entorno de red aislado para el contenedor y utilizando el módulo `slirp` del kernel para realizar la traducción de direcciones de red (NAT), lo que permite que el contenedor acceda a internet a través de la conexión de red del host.

## Ejemplo de uso de red en contenedores rootless

La primera limitación que tenemos a la hora de trabajar con contenedores rootless desde el punto de vista de la red, es que los usuarios no privilegiados no pueden usar puerto privilegiados (menores que 1024). 

```
$ podman run -dt --name webserver -p 80:80 quay.io/libpod/banner
Error: rootlessport cannot expose privileged port 80, you can add 'net.ipv4.ip_unprivileged_port_start=80' to /etc/sysctl.conf (currently 1024), or choose a larger port number (>= 1024): listen tcp 0.0.0.0:80: bind: permission denied
```

Podríamos cambiar ese comportamiento cambiando con `sysctl` el valor de `net.ipv4.ip_unprivileged_port_start` que por defecto tiene el valor de 1024.

```
sysctl net.ipv4.ip_unprivileged_port_start
net.ipv4.ip_unprivileged_port_start = 1024
```

Volvemos a crear el contenedor, teniendo en cuenta lo anterior:

```
$ podman run -dt --name webserver -p 8080:80 quay.io/libpod/banner
```

Y comprobamos su configuración de red:

```
$ podman exec webserver ip a
...
2: tap0: <BROADCAST,UP,LOWER_UP> mtu 65520 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 86:b3:30:2a:85:0e brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.100/24 brd 10.0.2.255 scope global tap0

$ podman exec webserver ip r
default via 10.0.2.2 dev tap0 
...
```
Vemos que se ha creado una interfaz virtual `tap0` con dirección 10.0.2.100 y puerta de enlace 10.0.2.2, que nos proporciona una conexión con la red del host, para permitir que este contenedor tenga conectividad con el exterior.

A continuación, vamos a crear un nuevo contenedor y volvemos a comprobar su configuración de red:

```
$ podman run -it --name cliente alpine
/ # ip a
...
2: tap0: <BROADCAST,UP,LOWER_UP> mtu 65520 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 12:ef:67:bd:21:1f brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.100/24 brd 10.0.2.255 scope global tap0
/ # ip r
default via 10.0.2.2 dev tap0 
```

Como podemos comprobar dicho contenedor tiene la misma configuración de red que el anterior. Los dos contenedores utilizan el mismo **espacio de nombres de red**, como consecuencia **los contenedores están completamente aislados unos de otros**, por lo que tendrán que utilizar los puertos expuestos para comunicarse y la dirección IP del host (en mi caso la 10.0.0.67):

```
/ # apk add curl
/ # curl http://10.0.0.67:8080
   ___          __              
  / _ \___  ___/ /_ _  ___ ____ 
 / ___/ _ \/ _  /  ' \/ _ `/ _ \
/_/   \___/\_,_/_/_/_/\_,_/_//_/
```