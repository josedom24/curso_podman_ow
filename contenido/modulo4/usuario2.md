# Uso de la red bridge definidas por el usuario

En este apartado vamos a trabajar con las dos redes que hemos creado en el apartado anterior.
En primer lugar vamos a crear dos contenedores conectados a la primera red, para ello usaremos el parámetro `--network` en el comando `podman run`:

```bash
$ sudo podman run -d --name servidorweb --network red1 nginx
$ sudo podman run -it --name cliente --network red1 alpine
```

Lo primero que vamos a comprobar es la resolución DNS desde el contenedor `cliente`:

```bash
# nslookup servidorweb
Server:		10.89.0.1
Address:	10.89.0.1:53

Non-authoritative answer:
Name:	servidorweb.dns.podman
Address: 10.89.0.2
```

Tenemos un servidor DNS en la dirección IP `10.89.0.1` que nos resuelve el nombre del primer contenedor en el dominio `dns.podman` con su dirección IP. Podemos comprobar que ese servidor DNS es el que tiene configurado el contenedor:

```bash
# cat /etc/resolv.conf 
search dns.podman
nameserver 10.89.0.1
```

Y por lo tanto podemos realizar conexiones usando el nombre de los contenedores:

```bash
# ping servidorweb
PING servidorweb (10.89.0.2): 56 data bytes
64 bytes from 10.89.0.2: seq=0 ttl=42 time=0.342 ms
...
```

## Conectando los contenedores a otras redes

A continuación, creamos un contenedor conectado a la segunda red (`red2`):

```bash
$ sudo podman run -it --name cliente2 --network red2 alpine
```
Comprobemos ahora la conectividad entre los contenedores, para ello obtenemos su dirección IP y su puerta de enlace, e intentamos acceder a uno de los contenedores creados anteriormente:


```bash
# ip a
2: eth0@if95: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether 6a:62:12:a1:24:f9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.1/24 brd 192.168.0.255 scope global eth0
...

# ip r
default via 192.168.0.100 dev eth0  metric 100 
...

# ping servidorweb
ping: bad address 'servidorweb'
```

Veamos cómo podemos conectar un contenedor a una red. Para ello usaremos el comando `podman network connect` y para desconectarla usaremos `podman network disconnect`. Salimos del contenedor que acabamos de crear, lo iniciamos y lo conectamos a la primera red:

```bash
$ sudo podman start cliente2
$ sudo podman network connect red1 cliente2
$ sudo podman attach cliente2
```

Comprobamos que se ha creado una nueva interfaz de red con el direccionamiento de la red `red1`:

```bash
# ip a
...
2: eth0@if97: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether aa:2d:36:83:57:27 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.2/24 brd 192.168.0.255 scope global eth0
...
3: eth1@if98: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether fa:c8:b2:6c:a8:b6 brd ff:ff:ff:ff:ff:ff
    inet 10.89.0.4/24 brd 10.89.0.255 scope global eth1
...
```

Ahora podemos comprobar si tenemos conectividad con el contenedor `servidorweb`:

```bash
# ping servidorweb
PING servidorweb (10.89.0.2): 56 data bytes
64 bytes from 10.89.0.2: seq=0 ttl=42 time=0.413 ms
...
```

Finalmente podemos desconectar el contenedor de la red, ejecutando el siguiente comando:

```bash
$ sudo podman network disconnect red1 cliente2
```

## Creación de Linux Bridge en el host

Como indicábamos anteriormente, al conectar contenedores a una determinada red se ha crea en el host un *Linux Bridge* que utiliza esa red. En nuestro caso, al estar trabajando con dos redes, se han creado dos *Linux Bridge*, podemos verlo ejecutando en el host la siguiente instrucción:

```bash
$ ip a
4 : podman1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 16:c6:77:f4:e1:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.89.0.1/24 brd 10.89.0.255 scope global podman1
...
6: podman2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 26:30:61:d7:de:09 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.100/24 brd 192.168.0.255 scope global podman2
...
```
Vemos que la puerta de enlace que han recibidos los contenedores conectados a estas redes corresponden con la dirección IP del host en el bridge correspondiente.

## Más opciones al trabajar con redes en docker

Tanto al crear un contenedor con el parámetro `--network` para conectarlo a una red, como con la instrucción `podman network connect`, podemos usar algunos otros parámetros.

Veamos un ejemplo donde vamos a crear un contenedor en la red `red2` que tenemos creada:

```bash
$ sudo podman run -it --name contenedor --network red2 \
                                   --ip 192.168.0.10 \
                                   --add-host=testing.example.com:192.168.0.20 \
                                   --dns 1.1.1.1 \
                                   --hostname servidor1 \
                                   alpine
```

* `--hostname servidor1`: Indicamos el nombre de la máquina. Lo comprobamos:

    ```bash
    # cat /etc/hostname 
    servidor1
    ```
* `--ip 192.168.0.10`: Nos permite poner una dirección IP fija en el contenedor. Vamos a comprobarlo:

    ```bash
    # ip a
    ...
    2: eth0@if100: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether b2:59:f5:78:61:68 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.10/24 brd 192.168.0.255 scope global eth0
    ...
    ```
* `--add-host=testing.example.com:192.168.100.20`: Añadimos un nuevo nombre de host como resolución estática. Lo comprobamos:

    ```bash
    # cat /etc/hosts
    ...
    192.168.0.20	testing.example.com
    ...
    
    # ping testing.example.com
    PING testing.example.com (192.168.0.20): 56 data bytes
    ```
* `--dns 1.1.1.1`: Hemos configurado como DNS el servidor `1.1.1.1`. Veamos esto con detenimiento, como hemos visto anteriormente al conectar el contenedor a una red bridge definida por el usuario se crea un servidor DNS que nos permite la resolución por el nombre del contenedor veamos el servidor DNS:
    ```bash
    # cat /etc/resolv.conf 
    search dns.podman
    nameserver 192.168.0.100
    ...
    ```
    Por defecto este servidor hace forward con el servidor DNS que tenga configurado el anfitrión (es decir usa el DNS del anfitrión para resolver los nombre que no conoce). Con la opción `--dns 1.1.1.1`, estamos cambiando el DNS al que hacemos forwarding, por lo tanto ese cambio no se visualiza en el fichero `/etc/resolv.conf`.
