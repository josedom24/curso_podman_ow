# Despliegue de la aplicación Temperaturas

En este ejemplo vamos a desplegar con Compose la aplicación Temperaturas, que estudiamos en un módulo anterior.

Puedes encontrar el fichero `compose.yaml` en el [Repositorio con el código de los ejemplos](xxx).

## Despliegue con contenedores rootful

En este caso el fichero `compose.yaml` puede tener este contenido:

```yaml
version: '3.1'
services:
  frontend:
    container_name: temperaturas-frontend
    image: docker.io/iesgn/temperaturas_frontend
    restart: always
    ports:
      - 8081:3000
    environment:
      TEMP_SERVER: temperaturas-backend:5000
    depends_on:
      - backend
  backend:
    container_name: temperaturas-backend
    image: docker.io/iesgn/temperaturas_backend
    restart: always
```

Veamos algunas observaciones:

* Aunque ya sabemos que la variable de entorno `TEMP_SERVER` tiene el valor `temperaturas-backend:5000` por defecto, la hemos indicado para configurar el nombre del contenedor redis.
* Podríamos haber usado también el nombre del servicio, es decir, `TEMP_SERVER: backend:5000`, ya que, como hemos comentado, la resolución se puede hacer usando el nombre del contenedor o el nombre del servicio.

Para crear el escenario:

```
$ sudo podman-compose up -d
```

Para listar los contenedores:

```
$ sudo podman-compose ps
CONTAINER ID  IMAGE                                         COMMAND         CREATED        STATUS        PORTS                   NAMES
f933fa3772c4  docker.io/iesgn/temperaturas_backend:latest   python3 app.py  2 minutes ago  Up 2 minutes                          temperaturas-backend
1134baf76caa  docker.io/iesgn/temperaturas_frontend:latest  python3 app.py  2 minutes ago  Up 2 minutes  0.0.0.0:8081->3000/tcp  temperaturas-frontend
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
  frontend:
    container_name: temperaturas-frontend
    image: docker.io/iesgn/temperaturas_frontend
    network_mode: "slirp4netns:port_handler=slirp4netns"
    restart: always
    ports:
      - 8081:3000
    environment:
      TEMP_SERVER: 10.0.0.231:5000
      NETWORK_INTERFACE: tap0
    depends_on:
      - backend
  backend:
    container_name: temperaturas-backend
    image: docker.io/iesgn/temperaturas_backend
    network_mode: "slirp4netns:port_handler=slirp4netns"
    restart: always
    ports:
      - 5000:5000
    environment:
      NETWORK_INTERFACE: tap0
```

Veamos algunas observaciones:

* En este ejemplo la dirección IP del host es `10.0.0.231`.
* El valor de la variable de configuración `TEMP_SERVER` para configurar el microservicio `frontend` para indicarle donde tiene que conectar al microservicio `backend` debe valer la dirección IP del host y el puerto 5000/tcp.
* En los dos contenedor debemos mapear el puerto: en el contenedor `frontend` porque vamos acceder desde el exterior (recordando que no podemos usar puertos privilegiados), y en el contenedor `backend` porque se va a acceder desde el otro contenedor.
* Hemos indicado el modo de red con el parámetro `network_mode` y el valor `slirp4netns:port_handler=slirp4netns` para indicar que utilice la red de tipo slirp4netns para realizar la conexión.
* También hemos creado una variable de entorno `NETWORK_INTERFACE` para indicar el nombre del dispositivo tap que se va a utilizar.

Levantamos el escenario ejecutando:

```
$ podman-compose up -d
```

Y podemos acceder con el navegador web a la aplicación y probar su funcionamiento.