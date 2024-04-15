# Ejecución de contenedores demonios

En esta ocasión hemos utilizado la opción `-d` del comando `podman run`, para que la ejecución del comando en el contenedor se haga en segundo plano, de manera desatendida, sin estar conectada a la entrada y salida estándar.

Por ejemplo, en este ejemplo ejecutamos un proceso demonio en el contenedor (proceso que se ejecuta indefinidamente) y utilizamos la poción `-d` para que se ejecute de manera desatendida, en segundo planto. En el ejemplo ejecutamos un bucle infinito que va escribiendo secuencialmente número y los escribe en un fichero cada un segundo:

```
$ podman run -d --name contenedor1 ubuntu bash -c 'i=0;while true; do echo $((i=i+1)); echo $i >> numeros.txt; sleep 1; done'
```

En la instrucción `podman run` hemos ejecutado el comando con `bash -c` que nos permite ejecutar uno o más comandos en el contenedor de forma más compleja (por ejemplo, indicando ficheros dentro del contenedor).

Comprobamos que el contenedor se está ejecutando:

```
$ podman ps
CONTAINER ID  IMAGE                            COMMAND               CREATED        STATUS        PORTS       NAMES
7a3e97be08c1  docker.io/library/ubuntu:latest  bash -c i=0;while...  3 minutes ago  Up 3 minutes              contenedor1
```

Podemos visualizar los logs del contenedor, ejecutando el siguiente comando:

```
$ sudo podman logs contenedor1
```

Con la opción `logs -f` seguimos visualizando los logs en tiempo real.

## Ciclo de vida de los contenedores

Tenemos varios comandos que nos permiten controlar el ciclo de vida del contenedor. En una terminar podemos visualizar los logs del contenedor y en otro terminar ejecutar los comando para ver como se comporta el contenedor:

* `podman start`: Inicia la ejecución de un contenedor que está parado.
* `podman stop`: Detiene la ejecución de un contenedor en ejecución.
* `podman restart`: Para y vuelve a iniciar la ejecución de un contenedor.
* `podman pause`: Pausa la ejecución de un contenedor.
* `podman unpause`: Continúa la ejecución de un contenedor que estaba pausado..

## Ejecución de comandos en contenedores

Si tenemos un contenedor que está iniciado, podemos ejecutar comandos en él con `podman exec`. Por ejemplo:

```
$ podman exec contenedor1 ls
...
nuemeros.txt
...
$ podman exec contenedor1 cat numeros.txt
```

Si queremos acceder interactivamente al contenedor, podemos ejecutar:

```
$ podman exec -it contenedor1 bash
```

## Copiar ficheros en contenedores

Con el comando `podman cp` podemos copiar ficheros a o desde un contenedor. Por ejemplo, si tengo un fichero en mi equipo lo puedo copiar al contenedor:

```
$ echo "Curso Podman">podman.txt
$ podman cp podman.txt contenedor1:/tmp
```

Podemos comprobar que el fichero existe en el contenedor:

```
$ podman exec contenedor1 cat /tmp/podman.txt
Curso Podman
```

Evidentemente, también podemos copiar ficheros desde el contenedor a nuestro equipo:

```
$ podman cp contenedor1:numeros.txt .
```

## Ejecución de un contenedor demonio con un servidor web

En este ejemplo vamos a crear un contenedor demonio que ejecuta un servidor web Apache, para ello vamos a usar la imagen `httpd:2.4` del registro **Docker Hub** (en este caso hemos indicado el nombre de la imagen y su etiqueta `2.4` que nos indica la versión del servidor web que vamos a usar). Podemos crear un contenedor rootful:

```
$ sudo podman run -d --name my-apache-app -p 80:80 docker.io/httpd:2.4
```

O podemos crear un contenedor rootless, teniendo en cuenta que no podemos utilizar los puertos privilegiados (menores del 1024), por lo que en este caso hemos mapeado el puerto 8080:

```
$ podman run -d --name my-apache-app -p 8080:80 docker.io/httpd:2.4
```

Hay que tener en cuenta que los contenedores que estamos creando se conectan a una red virtual privada y que toman direccionamiento dinámico. No solemos usar la dirección IP del contenedor para acceder al servicio que nos ofrece. Con la opción `-p` mapeamos un puerto del host, con el puerto del servicio ofrecido por el contenedor. Si accedemos a la dirección IP del ordenador que tiene instalado Podman al primer puerto indicado, se redireccionará la petición a la dirección IP del contenedor al segundo puerto indicado. **Nunca utilizamos directamente la dirección IP del contenedor para acceder a él**. 

Podemos ver los puertos que están mapeados en un contenedor de dos maneras distintas. Usando el comando `podman port`:

```
$ podman port my-apache-app
80/tcp -> 0.0.0.0:8080
```

O utilizando el comando `podman inspect` con un filtro:

```
$ podman inspect --format='{{range $p, $conf := .NetworkSettings.Ports}}  {{(index $conf 0).HostPort}} -> {{$p}} {{end}}' my-apache-app
```

Para probarlo accedemos desde un navegador web:

* Si estamos accediendo desde el host, accederemos a `http://localhost:8080`.
* Si estamos accediendo desde un ordenador remoto, accederemos a la dirección IP del host y al puerto, por ejemplo,si dirección IP del host es `172.22.201.251`, accederemos a `http://172.22.201.251:8080`:

![web](img/web.png)

Para acceder al log del contenedor podemos ejecutar:

```
$ podman logs my-apache-app
```

### Modificación del contenido servido por el servidor web

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


## Eliminar un contenedor demonio

Para eliminar un contenedor demonio, lo debemos para y posteriormente borrarlo:

```
$ podman stop contenedor1
$ podman rm contenedor1
```

Hay que tener en cuenta que un contenedor que esta ejecutándose no puede ser eliminado. Tendríamos que parar el contenedor y posteriormente borrarlo. Otra opción es borrarlo a la fuerza:

```
$ podman rm -f contenedor1
```
