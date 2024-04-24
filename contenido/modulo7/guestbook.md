# Despliegue de la aplicación Guestbook

En este ejemplo vamos a desplegar con Compose la aplicación Guestbook, que estudiamos en un módulo anterior.

Puedes encontrar los ficheros que vamos a utilizar en el directorio `modulo7/guestbook` del [Repositorio con el código de los ejemplos](https://github.com/josedom24/ejemplos_curso_podman_ow).

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
* Por último indicar que hemos usado un volumen llamado `redis` para guardar la información de la base de datos.

Para crear el escenario:

```
$ podman-compose up -d
```

Para listar los contenedores:

```
$ podman-compose ps
CONTAINER ID  IMAGE                             COMMAND               CREATED        STATUS        PORTS                   NAMES
ae891d324ba4  docker.io/iesgn/guestbook:latest  python3 app.py        5 seconds ago  Up 5 seconds  0.0.0.0:8080->5000/tcp  guestbook
c3563d156a15  docker.io/library/redis:latest    redis-server --ap...  4 seconds ago  Up 4 seconds                          redis
```

Podemos acceder desde el navegador web para comprobar que la aplicación está funcionando.

Para eliminar el escenario:

```
$ podman-compose down -v
```

