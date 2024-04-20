# Despliegue de la aplicación Guestbook

En este ejemplo vamos a desplegar con Compose la aplicación Guestbook, que estudiamos en un módulo anterior.

Puedes encontrar el fichero `compose.yaml` en el [Repositorio con el código de los ejemplos](xxx).

## Despliegue con contenedores rootful

El contenido del fichero `compose.yaml` es:

```yaml
version: '3.1'
services:
  app:
    container_name: guestbook
    image: docker.io/iesgn/guestbook
    restart: always
    environment:
      REDIS_SERVER: redis
    ports:
      - 8080:5000
  db:
    container_name: redis
    image: docker.io/redis
    restart: always
    command: redis-server --appendonly yes
    volumes:
      - redis:/data
volumes:
  redis:
```

Veamos algunas observaciones:

* Aunque ya sabemos que la variable de entorno `REDIS_SERVER` tiene el valor `redis` por defecto, la hemos indicado para configurar el nombre del contenedor redis.
* Podríamos haber usado también el nombre del servicio, es decir, `REDIS_SERVER: db`, ya que, como hemos comentado, la resolución se puede hacer usando el nombre del contenedor o el nombre del servicio.
* Como vimos cuando desplegamos esta aplicación en un módulo anterior, al crear el contenedor tenemos que ejecutar el comando `redis-server --appendonly yes` para que redis guarde la información de la base de datos en el directorio `/datos`. Para indicar el comando que hay que ejecutar al crear el contenedor usamos el parámetro `command`.
* Por último indicar que hemos usado un volumen docker llamado `redis` para guardar la información de la base de datos.

Para crear el escenario:

```
$ sudo podman-compose up -d
```

Para listar los contenedores:

```
$ sudo podman-compose ps
CONTAINER ID  IMAGE                             COMMAND               CREATED        STATUS        PORTS                   NAMES
ae891d324ba4  docker.io/iesgn/guestbook:latest  python3 app.py        5 seconds ago  Up 5 seconds  0.0.0.0:8080->5000/tcp  guestbook
c3563d156a15  docker.io/library/redis:latest    redis-server --ap...  4 seconds ago  Up 4 seconds                          redis
```

Podemos acceder desde el navegador web para comprobar que la aplicación está funcionando.

Para eliminar el escenario:

```
$ sudo podman-compose down -v
```

## Despliegue con contenedores rootless

El contenido del fichero `compose.yaml` es:

```yaml
version: '3.1'
services:
  app:
    container_name: guestbook
    image: docker.io/iesgn/guestbook
    restart: always
    ports:
      - 8080:5000
    network_mode: "slirp4netns:port_handler=slirp4netns"
    environment:
      REDIS_SERVER: 10.0.0.231
      NETWORK_INTERFACE: tap0
  db:
    container_name: redis
    image: docker.io/redis
    restart: always
    command: redis-server --appendonly yes
    ports:
      - 6379:6379
    volumes:
      - redis:/data
    network_mode: "slirp4netns:port_handler=slirp4netns"
    environment:
      NETWORK_INTERFACE: tap0
volumes:
  redis:
```

Veamos algunas observaciones:

* En este ejemplo la dirección IP del host es `10.0.0.231`.
* El valor de la variable de configuración `REDIS_SERVER` para configurar la aplicación Guestbook para indicarle donde tiene que conectar a la base de datos debe valer la dirección IP del host.
* En los dos contenedor debemos mapear el puerto: en el contenedor `guestbook` porque vamos acceder desde el exterior (recordando que no podemos usar puertos privilegiados), y en el contenedor `redis` porque se va a acceder desde el otro contenedor.
* Hemos indicado el modo de red con el parámetro `network_mode` y el valor `slirp4netns:port_handler=slirp4netns` para indicar que utilice la red de tipo slirp4netns para realizar la conexión.
* También hemos creado una variable de entorno `NETWORK_INTERFACE` para indicar el nombre del dispositivo tap que se va a utilizar.

Levantamos el escenario ejecutando:

```
$ podman-compose up -d
```

Y podemos acceder con el navegador web a la aplicación y probar su funcionamiento.
