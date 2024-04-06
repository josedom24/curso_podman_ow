# Ejemplo: Despliegue de WordPress + MariaDB en un Pod

Para la instalación de WordPress vamos a crear un Pod con dos contenedores: uno para ejecutar la base de datos MariaDB (imagen `docker.io/mariadb`) y el servidor web con la aplicación (imagen `docker.io/wordpress`). No es necesario conectar el Pod a un red definida por el usuario, ya que no necesitamos la característica de la resolución DNS, ya que la comunicación entre los contenedores de un Pod se hace a través de la interfaz loopback.

En primer lugar creamos el pod:

```
$ sudo podman pod create --name wordpress-pod -p 8080:80
```

Siguiendo la documentación de la imagen [`docker.io/mariadb`](https://hub.docker.com/_/mariadb) y la imagen [`docker.io/wordpress`](https://hub.docker.com/_/wordpress) podemos ejecutar los siguientes comandos para añadir los dos contenedores:

```
$ sudo podman run --pod wordpress-pod -d --name db \
                -v wpvol:/var/lib/mysql \
                -e MARIADB_DATABASE=wordpress \
                -e MARIADB_USER=wordpress \
                -e MARIADB_PASSWORD=wordpress \
                -e MARIADB_ROOT_PASSWORD=myrootpasswd \
                docker.io/mariadb

$ sudo podman  run --pod wordpress-pod -d --name wordpress \
                -v dbvol:/var/www/html \
                -e WORDPRESS_DB_HOST=127.0.0.1 \
                -e WORDPRESS_DB_USER=wordpress \
                -e WORDPRESS_DB_PASSWORD=wordpress \
                -e WORDPRESS_DB_NAME=wordpress \
                docker.io/wordpress
```

Vemos los Pods y contenedores que hemos creado:

```
$ sudo podman pod ps --ctr-names
POD ID        NAME        STATUS      CREATED         INFRA ID      NAMES
d1d937d15358  wordpress-pod      Running     3 minutes ago   27326c5e4a67  d1d937d15358-infra,db,wordpress

$ sudo podman ps --pod
CONTAINER ID  IMAGE                                    COMMAND               CREATED        STATUS        PORTS                 NAMES               POD ID        PODNAME
27326c5e4a67  localhost/podman-pause:4.9.4-1711445992                        3 minutes ago  Up 2 minutes  0.0.0.0:8080->80/tcp  d1d937d15358-infra  d1d937d15358  wordpress-pod
cd4aea32b99f  docker.io/library/mariadb:latest         mariadbd              3 minutes ago  Up 2 minutes  0.0.0.0:8080->80/tcp  db          d1d937d15358  wordpress-pod
bb71fd462b64  docker.io/library/wordpress:latest       apache2-foregroun...  2 minutes ago  Up 2 minutes  0.0.0.0:8080->80/tcp  wordpress         d1d937d15358  wordpress-pod
```

Y comprobamos que podemos acceder a la aplicación:

![wp](img/wp.png)