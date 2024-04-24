# Despliegue de la aplicación Temperaturas

En este ejemplo vamos a desplegar con Compose la aplicación Temperaturas, que estudiamos en un módulo anterior.

Puedes encontrar los ficheros que vamos a utilizar en el directorio `modulo7/temperaturas` del [Repositorio con el código de los ejemplos](https://github.com/josedom24/ejemplos_curso_podman_ow).

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

* Aunque ya sabemos que la variable de entorno `TEMP_SERVER` tiene el valor `temperaturas-backend:5000` por defecto, la hemos indicado para configurar el nombre del contenedor `backend`.
* Podríamos haber usado también el nombre del servicio, es decir, `TEMP_SERVER: backend:5000`, ya que, como hemos comentado, la resolución se puede hacer usando el nombre del contenedor o el nombre del servicio.

Para crear el escenario:

```
$ podman-compose up -d
```

Para listar los contenedores:

```
$ podman-compose ps
CONTAINER ID  IMAGE                                         COMMAND         CREATED        STATUS        PORTS                   NAMES
f933fa3772c4  docker.io/iesgn/temperaturas_backend:latest   python3 app.py  2 minutes ago  Up 2 minutes                          temperaturas-backend
1134baf76caa  docker.io/iesgn/temperaturas_frontend:latest  python3 app.py  2 minutes ago  Up 2 minutes  0.0.0.0:8081->3000/tcp  temperaturas-frontend
```

Podemos acceder desde el navegador web para comprobar que la aplicación está funcionando.

Para eliminar el escenario:

```
$ podman-compose down -v
```

