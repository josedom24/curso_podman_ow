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

# Ejemplo: Creando un contenedor con un servidor web

En este ejemplo vamos a crear un contenedor demonio que ejecuta un servidor web Apache, para ello vamos a usar la imagen `httpd:2.4` del registro **Docker Hub** (en este caso hemos indicado el nombre de la imagen y su etiqueta `2.4` que nos indica la versión del servidor web que vamos a usar):

```
$ sudo podman run -d --name my-apache-app -p 8080:80 docker.io/httpd:2.4
```

Hay que tener en cuenta que los contenedores que estamos creando se conectan a una red virtual privada y que toman direccionamiento dinámico. No solemos usar la dirección IP del contenedor para acceder al servicio que nos ofrece. Con la opción `-p` mapeamos un puerto del host, con el puerto del servicio ofrecido por el contenedor. Si accedemos a la dirección IP del ordenador que tiene instalado Podman al primer puerto indicado, se redireccionará la petición a la dirección IP del contenedor al segundo puerto indicado. **Nunca utilizamos directamente la dirección IP del contenedor para acceder a él**. 

Podemos ver los puertos que están mapeados en un contenedor de dos maneras distintas. Usando el comando `podman port`:

```
$ sudo podman port my-apache-app
80/tcp -> 0.0.0.0:8080
```

O utilizando el comando `podman inspect` con un filtro:

```
$ sudo podman inspect --format='{{range $p, $conf := .NetworkSettings.Ports}}  {{(index $conf 0).HostPort}} -> {{$p}} {{end}}' my-apache-app
```

Para probarlo accedemos desde un navegador web:

* Si estamos accediendo desde el host, accederemos a `http://localhost:8080`.
* Si estamos accediendo desde un ordenador remoto, accederemos a la dirección IP del host y al puerto, por ejemplo,si dirección IP del host es `172.22.201.251`, accederemos a `http://172.22.201.251:8080`:

![web](img/web.png)

Para acceder al log del contenedor podemos ejecutar:

```
$ sudo podman logs my-apache-app
```

## Modificación del contenido servido por el servidor web

Si consultamos la documentación de la imagen [`httpd`](https://hub.docker.com/_/httpd) en el registro Docker Hub, podemos determinar que el servidor web que se ejecuta en el contenedor guarda los ficheros que sirve (directorio *DocumentRoot*) en `/usr/local/apache2/htdocs/`. Vamos a crear un nuevo fichero `index.html` en ese directorio.

Lo podemos hacer de varias formas:

* Accediendo de forma interactiva al contenedor y haciendo la modificación:

    ```
    $ sudo podman exec -it my-apache-app bash

    root@cf3cd01a4993:/usr/local/apache2# cd /usr/local/apache2/htdocs/
    root@cf3cd01a4993:/usr/local/apache2/htdocs# echo "<h1>Curso Podman</h1>" > index.html
    root@cf3cd01a4993:/usr/local/apache2/htdocs# exit
    ```

* Ejecutando directamente el comando de creación del fichero `index.html` en el contenedor:

    ```
    $ sudo podman exec my-apache-app bash -c 'echo "<h1>Curso Podman</h1>" > /usr/local/apache2/htdocs/index.html'
    ```

* Usando el comando `podman cp` y copiando el fichero `index.html` al contenedor:

    ```
    $ echo "<h1>Curso Podman</h1>" > index.html
    $ sudo podman cp index.html  my-apache-app:/usr/local/apache2/htdocs/
    ```
    
Independientemente de cómo hayamos creado el fichero, podemos volver a acceder al servidor web y comprobar que efectivamente hemos cambiado el contenido del `index.html`:

![web](img/web2.png)