# Redes en contenedores rootful

Hasta ahora todos los contenedores rootful lo hemos conectado a la red **bridge** por defecto. Como ya hemos dicho las características más importantes de este tipo de red son las siguientes:
    
* Se crea en el host un *Linux Bridge* llamado **podman0**.
* El direccionamiento de esta red es 10.88.0.0/16.
* Usamos el parámetro `--publish` o `-p` en `podman run` para exponer algún puerto. Se crea una regla DNAT para tener acceso al puerto.
* Los contenedores conectados a un red **bridge** tiene acceso a internet por medio de una regla SNAT.

## Ejemplo de uso de la red bridge por defecto

Vamos a crear un contenedor rootful conectado a la red **bridge** por defecto a partir de la imagen `alpine`. Vamos a mapear el puerto 80/tcp ya que a continuación instalaremos un servidor web en el contenedor:

```bash
$ sudo podman run -it -p 8080:80 --name contenedor1 alpine ash
```

**Nota**: `ash` es la shell de la distribución Alpine. En realidad no habría que indicarlo, ya que el contenedor va a ejecutar ese comando por defecto.

### Configuración de red del contenedor

Accedemos al contenedor y comprobamos su configuración de red:

```bash
# ip a
...
2: eth0@if87: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether da:03:00:2b:57:44 brd ff:ff:ff:ff:ff:ff
    inet 10.88.0.55/16 brd 10.88.255.255 scope global eth0
    ...

# ip r
default via 10.88.0.1 dev eth0  metric 100
...
```

Hemos visto que su dirección IP está en la red `10.88.0.0/16` y la puerta de enlace es la `10.88.0.1` que corresponde a la dirección IP del host en esta red. Además podemos ver que la configuración DNS, que se guarda en el fichero `/etc/resolv.conf`, es la misma que la del host.

```bash
# cat /etc/resolv.conf
```

En el host podemos comprobar que se ha creado un Linux Bridge al que esta conectado el host y el contenedor, esta instrucción la podemos ejecutar en otra terminal:

```bash
$ ip a
...
3: podman0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether fe:50:31:ca:47:30 brd ff:ff:ff:ff:ff:ff
    inet 10.88.0.1/16 brd 10.88.255.255 scope global podman0
    ...
5: veth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master podman0 state UP group default qlen 1000
...
```

### Conectividad del contenedor

Podemos comprobar que el contenedor tiene acceso al exterior:

```bash
# ping www.podman.io
PING www.podman.io (185.199.109.153): 56 data bytes
64 bytes from 185.199.109.153: seq=0 ttl=42 time=43.409 ms
```

Además si instalamos un servidor web podemos acceder utilizando el puerto que hemos mapeado:

```bash
# apk add apache2
# httpd -D foreground
```

Desde el host podemos probar el acceso:

```bash
$ curl http://localhost:8080
<html><body><h1>It works!</h1></body></html>
```

Podemos comprobar la configuración de cortafuegos que se ha configurado en el host. 

* Para permitir que los contenedores conectados a la red **bridge** por defecto tengan conectividad al exterior tenemos que hacer una regla NAT, más concretamente SNAT. 
* Cuando hemos mapeado el puerto 8080/tcp del host al puerto 80/tcp del contenedor, se ha creado una regla NAT, en concreto DNAT, que hace que todas las peticiones al puerto 8080/tcp del host se redirijan al puerto 80/tcp del contenedor. Veamos estas reglas iptables, en el host ejecutando:

Si inicializamos el cortafuegos es posible que las reglas que ha configurado podman se pierdan. Podemos ejecutar la instrucción `podman network reload` para volver a configurar el cortafuegos de forma adecuada.

## Mapeo de puertos

Vamos a ver distintas opciones para mapear los puertos en las creación de un contenedor. Como sabemos usamos el parámetro `-p` o `--publish` en el comando `podman run`, veamos algunos ejemplos de las configuraciones que podemos hacer:

* `-p 8080:80`: Mapea el puerto 8080/tcp en el host al puerto 80/tcp en el contenedor.
* `-p 192.168.1.100:8080:80`: Asigna el puerto 8080/tcp en el host accediendo a la IP `192.168.1.100` al puerto 80/tcp en el contenedor.
* `-p 8080:80/udp`: Asigna el puerto 8080/tcp del host al puerto 80/udp del contenedor.
* `-p 8080:80/tcp -p 8080:80/udp`: Mapea el puerto 8080/tcp en el host al puerto 80/tcp en el contenedor, y mapea el puerto 8080/udp en el host al puerto 80/udp en el contenedor.

Por ejemplo este contenedor sólo sería accesible desde el host:

```bash
$ sudo podman run -d -p 127.0.0.1:8081:80 --name contenedor2 nginx
```

## Conectividad entre los contenedores conectados a la red por defecto

Evidentemente los contenedores conectados a la red por defecto podrán comunicarse usando su dirección IP, sin embargo esta red no ofrece ningún mecanismo de DNS para que podamos conectarnos a otro contenedor usando su nombre. Veamos un ejemplo con los contenedores creados en este apartado:

En primer lugar vamos a averiguar que dirección IP ha tomado el `contenedor2`:

```bash
$ sudo podman inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' contenedor2
10.88.0.56
```

A continuación desde el `contenedor1`intentamos conectamos al segundo contenedor:

```bash
$ sudo podman start contenedor1
$ sudo podman attach contenedor1
/ # ping 10.88.0.56
PING 10.88.0.56 (10.88.0.56): 56 data bytes
64 bytes from 10.88.0.56: seq=0 ttl=42 time=0.587 ms
...

/ # ping contenedor2
ping: bad address 'contenedor2'
```

Como podemos observar tenemos conectividad desde el primer contenedor al segundo, pero sólo usando la dirección IP, no tengo un DNS que me permita conectarme al segundo contenedor utilizando su nombre.
