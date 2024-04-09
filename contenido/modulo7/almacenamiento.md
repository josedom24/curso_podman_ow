# Almacenamiento con Compose

## Definiendo volúmenes con podman-compose

Además de definir los servicios (parámetro `services`) en el fichero `compose.yaml`, podemos definir los volúmenes que vamos a necesitar en nuestra infraestructura. Además, como hemos visto, podremos indicar que volumen va a utilizar cada contenedor.

Veamos un ejemplo, puedes encontrar el fichero en el [Repositorio con el código de los ejemplos](...).

El contenido del fichero `compose.yaml` es:

```yaml
version: '3.1'
services:
  db:
    container_name: contenedor_mariadb
    image: docker.io/mariadb
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: asdasd
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
    mariadb_data:
```

Y podemos iniciar el escenario:

```bash
$ sudo podman-compose up -d

$ sudo podman-compose ps
CONTAINER ID  IMAGE                             COMMAND     CREATED         STATUS         PORTS       NAMES
29e293238197  docker.io/library/mariadb:latest  mariadbd    15 seconds ago  Up 14 seconds              contenedor_mariadb

```

Y comprobamos que se ha creado un nuevo volumen:

```bash
$ sudo podman volume ls
DRIVER      VOLUME NAME
local       mariadb_mariadb_data
...
```

En la definición del servicio `db` hemos indicado que el contenedor montará el volumen en un directorio determinado con el parámetro `volumes`. Podemos comprobar que efectivamente se ha realizado el montaje:

```bash
$ sudo podman inspect -f '{{json .Mounts}}' contenedor_mariadb
[{"Type":"volume","Name":"mariadb_mariadb_data","Source":"/var/lib/containers/storage/volumes/mariadb_mariadb_data/_data","Destination":"/var/lib/mysql","Driver":"local","Mode":"","Options":["nosuid","nodev","rbind"],"RW":true,"Propagation":"rprivate"}]
```

Recuerda que si necesitas iniciar el escenario desde 0, debes eliminar el volumen:

```bash
$ sudo podman-compose down -v
```

## Utilización de bind mount con podman-compose

De forma similar podemos indicar que un contenedor va a utilizar bind mount como almacenamiento. En este caso sería:

```yaml
version: '3.1'
services:
  db:
    container_name: contenedor_mariadb
    image: docker.io/mariadb
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: asdasd
    volumes:
      - ./data:/var/lib/mysql:Z
```

Y después de iniciar el escenario podemos ver cómo se ha creado el directorio `data`:

```bash
$ cd data/
/data$ ls
aria_log.00000001  aria_log_control  ibdata1  ib_logfile0  ibtmp1  mysql
```

Hay que tener en cuenta que si usamos bind mount, el comando `podman-compose down -v` no eliminará el directorio donde se guardan los datos, en este caso `./data`.