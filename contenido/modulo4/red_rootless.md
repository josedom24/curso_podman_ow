# Redes en contenedores rootless

Cuando trabajamos con contenedores rootless tenemos varios mecanismos para ofrecer conectividad al contenedor:

## Red slirp4netns

Es el mecanismo de red que se utiliza por defecto. 

El proyecto [**slirp4netns**](https://github.com/rootless-containers/slirp4netns) crea un entorno de red aislado para el contenedor y utilizando el módulo `slirp` del kernel para realizar la traducción de direcciones de red (NAT), lo que permite que el contenedor acceda a internet a través de la conexión de red del host.

Tenemos algunas limitaciones, la más importante es que los usuarios no privilegiados no pueden usar puertos privilegiados (menores que 1024). 

```
$ podman run -dt --name webserver -p 80:80 quay.io/libpod/banner
Error: rootlessport cannot expose privileged port 80, you can add 'net.ipv4.ip_unprivileged_port_start=80' to /etc/sysctl.conf (currently 1024), or choose a larger port number (>= 1024): listen tcp 0.0.0.0:80: bind: permission denied
```

Podríamos cambiar ese comportamiento cambiando con `sysctl` el valor de `net.ipv4.ip_unprivileged_port_start` que por defecto tiene el valor de 1024.

```
$ sysctl net.ipv4.ip_unprivileged_port_start
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

### Red por defecto en contenedores rootless con Podman 5.0

En la nueva versión de Podman, se ha cambiado el mecanismo de red de slirp4netns a [pasta](https://passt.top/passt/about/#pasta-pack-a-subtle-tap-abstraction). Este nuevo mecanismo ofrece mejor rendimiento y más funciones.

En este caso la configuración de red de todos los contenedores rootless sería la siguiente:

```
$ podman run -it --rm alpine ip a
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 65520 qdisc fq_codel state UNKNOWN qlen 1000
    link/ether 42:61:0e:89:08:41 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.40/24 brd 10.0.0.255 scope global noprefixroute eth0
...
$ podman run -it --rm alpine ip r
default via 10.0.0.1 dev eth0  metric 100 
10.0.0.0/24 dev eth0 scope link  metric 100 
...
```

## Red bridge por defecto

Como hemos visto anteriormente podemos conectar nuestros contenedores rootless a la red bridge por defecto:

```
$ podman run -d -p 8080:80 --network=podman --name contenedor1 quay.io/libpod/banner

$ $ podman exec -it contenedor1 ip a
...
2: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether ee:93:61:6b:47:ca brd ff:ff:ff:ff:ff:ff
    inet 10.88.0.3/16 brd 10.88.255.255 scope global eth0
...
```

Y comprobamos el acceso al servidor web:

```
$ curl http://localhost:8080
```

## Red bridge definida por el usuario

Un usuario sin privilegios también pueden definir sus propias redes bridge. Estas redes se crearán en el espacio de nombres de red del usuario:

```
$ podman network create mi_red

$ podman run -d -p 8081:80 --name servidorweb --network mi_red docker.io/nginx
$ podman run -it --name cliente --network mi_red alpine
```

Y comprobamos la conectividad entre contenedores usando su nombre:

```
# ping servidorweb
PING servidorweb (10.89.2.3): 56 data bytes
64 bytes from 10.89.2.3: seq=0 ttl=42 time=0.370 ms
...
```
Una característica que tenemos que tener en cuenta, es que esta nueva red se ha creado en el espacio de nombres de red del usuario, por lo tanto desde el host no tenemos conectividad con el contenedor:

```
$ ping 10.89.2.3
PING 10.89.2.3 (10.89.2.3) 56(84) bytes of data.
...
```

Sin embargo, si podemos acceder a la IP del host y al puerto que hemos mapeado para acceder a la aplicación:

```
$ curl http://localhost:8081
```

En este caso el *Linux Bridge* que se ha creado con la nueva red, no se ha creado en el espacio de red del host. Podemos comprobar que el bridge `podman3` no se ha creado en el host ejecutando `sudo ip a`.

Sin embargo, podemos acceder al espacio de nombres de red del usuario ejecutando la siguiente instrucción:

```
$ podman unshare --rootless-netns ip a
...
2: tap0: <BROADCAST,UP,LOWER_UP> mtu 65520 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether 76:eb:49:a9:64:2a brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.100/24 brd 10.0.2.255 scope global tap0
   ...
3: podman3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 1a:3a:83:8d:1e:b9 brd ff:ff:ff:ff:ff:ff
    inet 10.89.2.1/24 brd 10.89.2.255 scope global podman3
    ...
```

Con el parámetro `--rootless-netns` de `podman unshare` accedemos al espacio de nombres de red del usuario, donde comprobamos la interfaz de red de tipo TAP usada por slirp4netns, y los Linux Bridge que se van creando con cada una de las redes bridge definidas por el usuario sin privilegios.

Por ejemplo, accediendo al espacio de nombres de red del usuario, comprobamos que si tenemos conectividad con el contenedor:

```
$ podman unshare --rootless-netns ping 10.89.2.3
PING 10.89.2.3 (10.89.2.3) 56(84) bytes of data.
64 bytes from 10.89.2.3: icmp_seq=1 ttl=64 time=0.135 ms
...
```
## Red host

Este tipo conexión a red también lo podemos usar con contenedor rootless. Sin embargo, tenemos que tener en cuenta las limitaciones que tenemos al crear contenedores rootless, en este caso un usuario sin privilegios no puede usar puertos no privilegiados, por debajo del 1024. Por lo tanto vamos a usar una imagen de nginx que ejecuta el servidor nginx con un usuario sin privilegios y por lo tanto lo levanta en el puerto 8080/tcp.

Vamos a usar la imagen de nginx ofrecida por la empresa Bitnami, esta imagen tienen como característica que los procesos que se ejecutan al crear el contenedor son ejecutados por usuarios no privilegiados.

```
$ podman run -d --network host --name my_nginx docker.io/bitnami/nginx
```

Y podemos acceder al puerto 8080/tcp para comprobar que podemos acceder al servicio web.
