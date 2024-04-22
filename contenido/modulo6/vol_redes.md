# Gestionando volúmenes y redes con Systemd y Quadlet

En este ejemplo, vamos a gestionar un contenedor rootful ofreciendo un servidor de base de datos MariaDB. el contenedor será persistente usando un volumen y estará conectado a una red bridge definida por el usuario.

Para ello vamos a crear tres plantillas de unidad de Systemd, en el directorio `/etc/containers/systemd`.

Puedes encontrar los ficheros que vamos a utilizar en el directorio `modulo6/vol_redes` del [Repositorio con el código de los ejemplos](https://github.com/josedom24/ejemplos_curso_podman_ow).

En la plantilla `vol_mariadb.volume` definimos el volumen indicando el nombre:

```
[Volume]
VolumeName=vol-mariadb
```

En la plantilla `red_mariadb.network` definimos la red indicando su nombre, su direccionamiento y la dirección de la puerta de enlace:

```
[Network]
NetworkName=red-mariadb
Subnet=192.168.100.0/24
Gateway=192.168.100.1
```

Por último, en la plantilla `mariadb.container` definimos el contenedor que vamos a gestionar:

```
[Unit]
Description=Un contenedor con el servidor de base de datos MariaDB

[Container]
Image=docker.io/mariadb:10.5
ContainerName=contenedor_mariadb
Volume=vol_mariadb.volume:/usr/lib/mysql
Network=red_mariadb.network
Environment=MARIADB_ROOT_PASSWORD=my-secret-pw

[Service]
Restart=always
TimeoutStartSec=900

[Install]
# Start by default on boot
WantedBy=multi-user.target default.target
```

Como vemos indicamos la imagen. el nombre, el punto de montaje referenciando al volumen (indicando el nombre de la plantilla donde está definido el volumen), la red a la que está conectada (indicando el nombre de la plantilla donde está definida la red) y la variable de entorno para indicar la contraseña del usuario `root`.

A continuación, podemos iniciar el contenedor y comprobar los recursos que se han creado:

```
# systemctl daemon-reload

# systemctl start mariadb

# podman volume ls
DRIVER      VOLUME NAME
local       vol-mariadb

# podman network ls
NETWORK ID    NAME                         DRIVER
e913e24830f4  red-mariadb                  bridge

# podman ps
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS         PORTS                 NAMES
bb535bd81493  docker.io/library/mariadb:10.5  mysqld                16 minutes ago  Up 16 minutes                        contenedor_mariadb

# podman inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' contenedor_mariadb
192.168.100.2

# podman exec -it contenedor_mariadb bash -c "mariadb -u root -p -h 127.0.0.1"
Enter password: 
...
```
