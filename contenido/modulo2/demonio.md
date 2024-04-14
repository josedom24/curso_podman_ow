# Ejecución de contenedores demonios

En esta ocasión hemos utilizado la opción `-d` del comando `podman run`, para que la ejecución del comando en el contenedor se haga en segundo plano, de manera desatendida, sin estar conectada a la entrada y salida estándar.

```
$ sudo podman run -d --name contenedor4 ubuntu bash -c "while true; do echo hello world; sleep 1; done"
261ba787395294fff7c6515714d7148410eb6628d2db1cb781628d9dc00a0a48
```

> NOTA: En la instrucción `podman run` hemos ejecutado el comando con `bash -c` que nos permite ejecutar uno o más comandos en el contenedor de forma más compleja (por ejemplo, indicando ficheros dentro del contenedor).

Comprobamos que el contenedor se está ejecutando:

```
$ sudo podman ps
CONTAINER ID  IMAGE                            COMMAND               CREATED         STATUS         PORTS       NAMES
261ba7873952  docker.io/library/ubuntu:latest  bash -c while tru...  32 seconds ago  Up 32 seconds              contenedor4
```

Podemos visualizar los logs del contenedor, ejecutando el siguiente comando:

```
$ sudo podman logs contenedor4
```

Con la opción `logs -f` seguimos visualizando los logs en tiempo real.

Por último podemos parar el contenedor y borrarlo con las siguientes instrucciones:

```
$ sudo podman stop contenedor4
$ sudo podman rm contenedor4
```

Hay que tener en cuenta que un contenedor que esta ejecutándose no puede ser eliminado. Tendríamos que parar el contenedor y posteriormente borrarlo. Otra opción es borrarlo a la fuerza:

```
$ sudo podman rm -f contenedor4
```
## Ejecución de contenedores demonios rootless

Para crear un contenedor demonio, podemos ejecutar la siguiente instrucción:

```
$ podman run -d --name miweb -p 8080:80 quay.io/libpod/banner
```

* La imagen `quay.io/libpod/banner` es un servidor web con una página predeterminada. Es una imagen muy liviana.
* Al crear un contenedor rootless no podemos utilizar los puertos privilegiados (menores del 1024), por lo que hemos mapeado el puerto 8080.

Para comprobar que funciona, podemos acceder desde el mismo host. Por ejemplo, con la herramienta `curl`:

```
$ curl http://localhost:8080
   ___          __              
  / _ \___  ___/ /_ _  ___ ____ 
 / ___/ _ \/ _  /  ' \/ _ `/ _ \
/_/   \___/\_,_/_/_/_/\_,_/_//_/

```

## Configuración de contenedores con variables de entorno

Más adelante veremos que al crear un contenedor que necesita alguna configuración específica, lo que vamos a hacer es crear variables de entorno en el contenedor, para que el proceso que inicializa el contenedor pueda realizar dicha configuración.

Para crear una variable de entorno al crear un contenedor usamos el flag `-e` o `--env`:

```
$ podman run -it --name contenedor5 -e USUARIO=prueba ubuntu
root@145b1105bc62:/# echo $USUARIO
prueba
```