# Creación de contenedores rootless con podman-compose

Podemos crear contenedor rootless con `podman-compose`. Pero en este caso hay que indicar el tipo de red al que se conecta el contenedor, ya que por defecto `podman-compose` crea una red bridge definida por el usuario.

Tenemos que crear un fichero `compose.yaml` con la siguiente configuración:

```
version: '3.1'
services:
  app:
    network_mode: "slirp4netns:port_handler=slirp4netns"
    environment:
      - NETWORK_INTERFACE=tap0
    container_name: webserver
    image: quay.io/libpod/banner
    restart: always
    ports:
      - 8085:80
```

* Hemos indicado el modo de red con el parámetro `network_mode` y el valor `slirp4netns:port_handler=slirp4netns` para indicar que utilice la red de tipo slirp4netns para realizar la conexión.
* También hemos creado una variable de entorno `NETWORK_INTERFACE` para indicar el nombre del dispositivo tap que se va a utilizar.


A continuación podemos levantar el escenario, si utilizar `sudo` ya que vamos a ejecutar `podman-compose` con un usuario sin privilegios:

```
$ podman-compose up -d

$ podman-compose ps
CONTAINER ID  IMAGE                         COMMAND               CREATED         STATUS         PORTS                 NAMES
8dc9bf2d75f2  quay.io/libpod/banner:latest  nginx -g daemon o...  1 minute ago    Up 1 minute    0.0.0.0:8085->80/tcp  webserver
```

Y comprobamos el acceso:

```
$ curl http://localhost:8085
   ___          __              
  / _ \___  ___/ /_ _  ___ ____ 
 / ___/ _ \/ _  /  ' \/ _ `/ _ \
/_/   \___/\_,_/_/_/_/\_,_/_//_/
```
