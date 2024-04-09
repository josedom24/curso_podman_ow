# Almacenamiento y redes con Compose

Además de definir los servicios (parámetro `services`) en el fichero `compose.yaml`, podemos definir los volúmenes que vamos a necesitar en nuestra infraestructura y las redes. Tenemos que tener en cuenta los siguientes aspectos:

* En la sección `volumes` podemos indicar la creación de un volumen.
* En la definición del servicio podemos indicar el uso del volumen en un punto de montaje o el uso de un bind mount.
* Cuando creamos un escenario con `podman-compose` **se crea una nueva red bridge definida por el usuario donde se conectan los contenedores**, por lo tanto, obtenemos resolución por DNS que resuelve tanto el nombre del contenedor, como el nombre del servicio.
* El nombre de la red que se crea por defecto, será el nombre del proyecto (el indicado en el parámetro `name`o el nombre del directorio donde se almacena el fichero `compose.yaml` y la palabra `default`).
* Podemos indicar el nombre de la red en la sección `networks`. Opcionalmente es posible configurar dicha red.
* Hay que recordar que al ejecutar `podman-compose down` la red que se ha creado no se elimina.

Veamos un ejemplo, puedes encontrar el fichero en el [Repositorio con el código de los ejemplos](...).

En el directorio donde tenemos el fichero compose, hemos creado un directorio `data` con un fichero `fichero.txt`, para montarlo como un bind mount:

```
$ mkdir data
$ echo "Curso Podman" > fichero.txt
```

El contenido del fichero `compose.yaml` es:

```yaml
version: '3.1'
services:
  c1:
    container_name: contenedor1
    image: alpine
    tty: true
    restart: always
    networks:
      - red_externa
      - red_interna
    volumes:
      - volumen1:/data/volumen
      - ./data:/data/directorio:Z
    hostname: contenedor1
    command: ash
<<<<<<< HEAD
  c2:
    container_name: contenedor2
    image: alpine
    tty: true
    restart: always
    networks:
      - red_externa
    hostname: contenedor2
    command: ash
=======
>>>>>>> e732e804eceb25af2dd2a06b117b001f7a5e1712

networks:
    red_externa:
        ipam:
            config:
              - subnet: 192.168.10.0/24
                gateway: 192.168.10.1
    red_interna:
volumes:
  volumen1:
```

Algunos parámetros interesantes que encontramos en la definición del escenario:

* `tty: true`: Nos permiten abrir una terminal en el contenedor. Igual que la opción `-t` en la instrucción `podman run`.
* `command: ash`: Hemos indicado el comando que queremos ejecutar, aunque no sería necesario, ya que la imagen `añpine` ejecuta `ash` por defecto.
* `hostname: contenedor1`: HHemos indicado el hostname de la máquina.
* En el parámetro `volumes` de la definición del servicio indicamos los puntos de montajes. En este caso el primero es con el volumen que hemos creado y el segundo es un bind mount.
* Con el parámetro `networks` en la definición de los contenedores, hemos indicado la conexión del contenedor a las redes.
* La sección `networks` a continuación de la definición de servicios nos permite configurar las redes que vamos a usar en el escenario:
    * Indicamos el nombre de las redes que vamos a crear.
    * Configuramos la red en el parámetro `IPAM` y `config`, por ejemplo indicando el direccionamiento con el parámetro `subnet` y la puerta de enlace de la red con el parámetro `gateway`. En este caso, hemos configurado la primera red, pero no la segunda.

Y podemos iniciar el escenario:

```bash
$ sudo podman-compose up -d

$ sudo podman-compose ps
CONTAINER ID  IMAGE                            COMMAND     CREATED         STATUS         PORTS       NAMES
5275641a7ceb  docker.io/library/alpine:latest  ash         28 seconds ago  Up 25 seconds              contenedor1
```

## Comprobamos el alamcenamiento

Podemos ver que se ha creado un volumen cuyo nombre será el nombre del proyecto (el indicado en el parámetro `name` o el nombre del directorio donde se guarda el fichero `compose.yaml`).

```bash
$ sudo podman volume ls
...
local       redes_volumen1
```

Podemos ver los puntos de montajes que hemos creado:

```bash
$ sudo podman inspect -f '{{json .Mounts}}' contenedor1
[{"Type":"volume","Name":"redes_volumen1","Source":"/var/lib/containers/storage/volumes/redes_volumen1/_data","Destination":"/data/volumen","Driver":"local","Mode":"","Options":["nosuid","nodev","rbind"],"RW":true,"Propagation":"rprivate"},{"Type":"bind","Source":"/home/fedora/compose/redes/.data","Destination":"/data/directorio","Driver":"","Mode":"","Options":["rbind"],"RW":true,"Propagation":"rprivate"}]
```

Y comprobamos que podemos escribir en el volumen y listar los ficheros del bind mount:

```
$ sudo podman-compose exec c1 touch /data/volumen/fichero2.txt
$ sudo podman-compose exec c1 cat /data/directorio/fichero.txt
Curso Podman
```

## Comprobación de la red

Podemos ver las redes que se han creado:

```
$ sudo podman network ls
NETWORK ID    NAME                        DRIVER
2f259bab93aa  podman                      bridge
accbf91a04cc  redes_red_externa           bridge
ba76cb63cc8e  redes_red_interna           bridge
```

Podemos ver que el nombre de la red está formado por el nombre del proyecto y el nombre que hemos indicado en la definición.

Finalmente podemos comprobar la configuración de red del contenedor:

```
$ sudo podman-compose exec c1 ip a
...
2: eth0@if280: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether f2:fa:ef:ea:41:42 brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.9/24 brd 192.168.10.255 scope global eth0
    ...
3: eth1@if282: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether aa:78:53:f1:32:18 brd ff:ff:ff:ff:ff:ff
    inet 10.89.2.4/24 brd 10.89.2.255 scope global eth1
```


Comprobamos que tenemos resolución DNS tanto con el nombre del servicio como con el nombre del contenedor:

```bash
$ sudo podman-compose exec c1 nslookup contenedor2
...
Non-authoritative answer:
Name:	contenedor2.dns.podman
Address: 192.168.10.11

$ sudo podman-compose exec c1 nslookup c2
...
Non-authoritative answer:
Name:	contenedor2.dns.podman
Address: 192.168.10.11
```

Y por último, comprobamos que hay conectividad:

```bash
sudo podman-compose exec c1 ping contenedor2
PING contenedor2 (192.168.10.11): 56 data bytes
64 bytes from 192.168.20.20: seq=0 ttl=64 time=0.072 ms
...
```

Recuerda que si necesitas iniciar el escenario desde 0, debes eliminar el volumen:

```bash
$ sudo podman-compose down -v
```

