# Ejemplo: Despliegue de WordPress + MariaDB en un Pod

Para la instalación de WordPress vamos a crear un Pod con dos contenedores: uno para ejecutar la base de datos MariaDB (imagen `docker.io/mariadb`) y el servidor web con la aplicación (imagen `docker.io/wordpress`). Los dos contenedores lo vamos a conectar a la misma red, aunque la comunicación entre los contenedores de un Pod se hace a través de la interfaz loopbak.

Además vamos a usar bind mount para hacer los contenedores persistente. en este caso, no hay almacenamiento compartido, cada contenedor va a tener su almacenamiento independiente.

```
$ sudo podman network create red_wp
```

A continuación vamos a crear el Pod:

```
$ sudo podman pod create --name pod-wp -p 8888:80 --network red_wp
```

Siguiendo la documentación de la imagen [`docker.io/mariadb`](https://hub.docker.com/_/mariadb) y la imagen [`docker.io/wordpress`](https://hub.docker.com/_/wordpress) podemos ejecutar los siguientes comandos para añadir los dos contenedores:

```
$ mkdir -p wp/data
$ mkdir -p wp/cms

$ sudo podman run --pod pod-wp -d --name servidor_mariadb \
                -v ${PWD}/wp/data:/var/lib/mysql:Z \
                -e MARIADB_DATABASE=bd_wp \
                -e MARIADB_USER=user_wp \
                -e MARIADB_PASSWORD=asdasd \
                -e MARIADB_ROOT_PASSWORD=asdasd \
                docker.io/mariadb

$ sudo podman  run --pod pod-wp -d --name servidor_wp \
                -v ${PWD}/wp/cms:/var/www/html:Z \
                -e WORDPRESS_DB_HOST=127.0.0.1 \
                -e WORDPRESS_DB_USER=user_wp \
                -e WORDPRESS_DB_PASSWORD=asdasd \
                -e WORDPRESS_DB_NAME=bd_wp \
                docker.io/wordpress
```

Vemos los Pods y contenedores que hemos creado:

```
$ sudo podman pod ps --ctr-names
POD ID        NAME        STATUS      CREATED         INFRA ID      NAMES
d1d937d15358  pod-wp      Running     3 minutes ago   27326c5e4a67  d1d937d15358-infra,servidor_mariadb,servidor_wp

$ sudo podman ps --pod
CONTAINER ID  IMAGE                                    COMMAND               CREATED        STATUS        PORTS                 NAMES               POD ID        PODNAME
27326c5e4a67  localhost/podman-pause:4.9.4-1711445992                        3 minutes ago  Up 2 minutes  0.0.0.0:8888->80/tcp  d1d937d15358-infra  d1d937d15358  pod-wp
cd4aea32b99f  docker.io/library/mariadb:latest         mariadbd              3 minutes ago  Up 2 minutes  0.0.0.0:8888->80/tcp  servidor_mariadb    d1d937d15358  pod-wp
bb71fd462b64  docker.io/library/wordpress:latest       apache2-foregroun...  2 minutes ago  Up 2 minutes  0.0.0.0:8888->80/tcp  servidor_wp         d1d937d15358  pod-wp
```

Y comprobamos que podemos acceder a la aplicación:

![wp](img/wp.png)