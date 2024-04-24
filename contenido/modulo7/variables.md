# Uso de parámetros con Compose

* Nuestro fichero `compose.yaml` se puede parametrizar. Determinados datos se pueden poner con una variable a la que daremos un valor en el momento de creación del escenario.
* La ventaja de parametrizar el fichero `compose.yaml` es que nos permite, con un mismo fichero, desplegar nuestras aplicaciones en diferentes entornos. Por ejemplo, en despliegues en el entorno de desarrollo tendremos unos valores para las variables, y en el despliegue en el entorno de producción tendremos otro conjunto de valores.
* Las variables tienen la forma de `clave=valor` y se guardan en un fichero llamado `.env`.
* Para utilizar las variables en el fichero `compose.yaml` utilizaremos la sintaxis `${clave}`.

Puedes encontrar los ficheros que vamos a utilizar en el directorio `modulo7/variables` del [Repositorio con el código de los ejemplos](https://github.com/josedom24/ejemplos_curso_podman_ow).

Por ejemplo podríamos parametrizar el despliegue de WordPress + MariaDB utilizando las siguientes variables en el fichero `compose.yaml`:

```yaml
version: '3.1'
services:
  wordpress:
    container_name: servidor_wp
    image: docker.io/wordpress:${VERSION_WP}
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: ${USUARIO}
      WORDPRESS_DB_PASSWORD: ${PASS}
      WORDPRESS_DB_NAME: ${BASEDEDATOS}
    ports:
      - ${PUERTO}:80
    volumes:
      - wordpress_data:/var/www/html
  db:
    container_name: servidor_mysql
    image: docker.io/mariadb:${VERSION_MDB}
    restart: always
    environment:
      MARIADB_DATABASE: ${BASEDEDATOS}
      MARIADB_USER: ${USUARIO}
      MARIADB_PASSWORD: ${PASS}
      MARIADB_ROOT_PASSWORD: ${PASS_ROOT}
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
    wordpress_data:
    mariadb_data:
```

Como puedes observar hemos parametrizado las versiones de la imágenes (`VERSION_WP` y `VERSION_MDB`), el puerto de acceso (`PUERTO`) y las credenciales de acceso a la base de datos (`USUARIO`, `PASS`,`PASS_ROOT` y `BASEDEDATOS`).

Al ejecutar `podman-compose up` se busca en el mismo directorio donde está el fichero `compose.yaml` un fichero llamado `.env` con la declaración de las variables que vamos a usar. 


## Despliegue de la aplicación en el entorno de desarrollo

En el entorno de desarrollo para desplegar el escenario podríamos tener un fichero `.env` con los siguientes valores:

```
VERSION_WP=latest
VERSION_MDB=latest
PUERTO=8080
USUARIO="prueba"
PASS="asdasd"
PASS_ROOT="asdasd"
BASEDEDATOS="wordpress"
```

Desplegamos el escenario:

```
$ podman-compose up -d
```

Y podemos comprobar la configuración que hemos desplegado:

```
$ podman-compose ps
CONTAINER ID  IMAGE                               COMMAND               CREATED         STATUS         PORTS                 NAMES
13eb418d6b92  docker.io/library/wordpress:latest  apache2-foregroun...  28 seconds ago  Up 26 seconds  0.0.0.0:8080->80/tcp  servidor_wp
378c4e69e12a  docker.io/library/mariadb:latest    mariadbd              23 seconds ago  Up 21 seconds                        servidor_mysql
```

Vemos las versiones de las imágenes que hemos desplegado (`latest`), el puerto que hemos mapeado (el 8080/tcp) y podemos ver las variables de entorno que se han creado, por ejemplo, en el contenedor de la base de datos:

```
$ podman-compose exec db env
MARIADB_USER=prueba
MARIADB_PASSWORD=asdasd
MARIADB_ROOT_PASSWORD=asdasd
MARIADB_DATABASE=wordpress
...
```

## Despliegue de la aplicación en el entorno de producción

En este caso en el fichero `.env` tenemos definidas las siguientes variables:

```
VERSION_WP=php8.3-apache
VERSION_MDB=10.5
PUERTO=80
USUARIO="user_server_1345"
PASS="0sFPBmHeDvgu5DOpACFsQ5MhH1J"
PASS_ROOT="4KUHGOa1CWciYopkAw9eBZdBtbu"
BASEDEDATOS="wp_server_bd"
```

Realizamos el despliegue:

```
$ podman-compose up -d
```

Y podemos comprobar la configuración que hemos desplegado:

```
$ podman-compose ps
CONTAINER ID  IMAGE                                      COMMAND               CREATED         STATUS         PORTS               NAMES
cd1a229b336e  docker.io/library/wordpress:php8.3-apache  apache2-foregroun...  18 seconds ago  Up 15 seconds  0.0.0.0:80->80/tcp  servidor_wp
4ce872c294a2  docker.io/library/mariadb:10.5             mysqld                10 seconds ago  Up 8 seconds                       servidor_mysql
```

Vemos las versiones de las imágenes que hemos desplegado (`mariadb:10.5` y `wordpress:php8.3-apache`), el puerto que hemos mapeado (el 80/tcp) y podemos ver las variables de entorno que se han creado, por ejemplo, en el contenedor de la base de datos:

```
$ podman-compose exec db env
MARIADB_USER=user_server_1345
MARIADB_PASSWORD=0sFPBmHeDvgu5DOpACFsQ5MhH1J
MARIADB_ROOT_PASSWORD=4KUHGOa1CWciYopkAw9eBZdBtbu
MARIADB_DATABASE=wp_server_bd
...
```



