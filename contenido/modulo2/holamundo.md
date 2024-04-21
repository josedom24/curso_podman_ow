# El "Hola Mundo" de Podman

Vamos a crear nuestro primer contenedor, para comprobar que todo está funcionando y vamos a explicar el proceso que se va a realizar en la creación del contenedor.

## Contenedor rootful

Vamos a crear un contenedor **rootful**, es decir un contenedor ejecutado por el usuario `root`, por eso usaremos la instrucción `sudo`:

```
$ sudo podman run hello-world
Resolved "hello-world" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull quay.io/podman/hello:latest...
Getting image source signatures
Copying blob 83d537244900 done   | 
Copying config a4e07799a3 done   | 
Writing manifest to image destination
!... Hello Podman World ...!

         .--"--.           
       / -     - \         
      / (O)   (O) \        
   ~~~| -=(,Y,)=- |         
    .---. /`  \   |~~      
 ~/  o  o \~~~~.----. ~~   
  | =(X)= |~  / (O (O) \   
   ~~~~~~~  ~| =(Y_)=-  |   
  ~~~~    ~~~|   U      |~~ 

Project:   https://github.com/containers/podman
Website:   https://podman.io
Desktop:   https://podman-desktop.io
Documents: https://docs.podman.io
YouTube:   https://youtube.com/@Podman
X/Twitter: @Podman_io
Mastodon:  @Podman_io@fosstodon.org
```

Pero, ¿qué es lo que está sucediendo al ejecutar esa orden?:

1. Indicamos que queremos ejecutar un contenedor (`podman run`).
2. Indicamos el nombre de la imagen. Hemos usado el nombre corto sin indicar el nombre del registro, ya que la imagen `hello-world` esta definida en el fichero `/etc/containers/registries.conf.d/000-shortnames.conf` donde indicamos alias a los nombres de las imágenes para que sea más fácil su gestión.
3. Al ser la primera vez que se ejecuta un contenedor basado en la imagen `hello-word`, la imagen se descarga de un registro público, en este caso del registro **quay.io** y se guarda en nuestro registro local. La etiqueta por defecto de la imagen es `latest`.
3. Se crea el contenedor que ejecuta un comando por defecto que muestra el mensaje que hemos leído y se detiene.

**NOTA**: En realidad todas los comandos del cliente Podman que trabajan con contenedores son subcomandos de `podman container`, pero se puede abreviar omitiendo el comando `container`, es decir estos dos comandos son iguales:

```
$ sudo podman container run ...
$ sudo podman run...
```

Si listamos los contenedores que se están ejecutando (`podman ps`):

```
$ sudo podman ps
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```

Comprobamos que este contenedor no se está ejecutando. **Un contenedor ejecuta un proceso y cuando termina la ejecución, el contenedor se para.**

Para ver los contenedores que no se están ejecutando (observa que se ha asignado un nombre aleatorio al contenedor), ejecutamos:

```
$ sudo podman ps -a
CONTAINER ID  IMAGE                            COMMAND               CREATED         STATUS                     PORTS       NAMES
03038845a764  quay.io/podman/hello:latest      /usr/local/bin/po...  3 minutes ago   Exited (0) 3 minutes ago               stupefied_mcnulty
```

Para eliminar el contenedor podemos identificarlo con su `id`:

```
$ sudo podman rm 03038845a764
```

o con su nombre:

```
$ sudo podman rm stupefied_mcnulty
```

La imagen que hemos usado se queda descargada en nuestro registro local:

```
$ sudo podman images
REPOSITORY            TAG         IMAGE ID      CREATED       SIZE
quay.io/podman/hello  latest      9e0dd3d2c11c  17 hours ago  752 kB
```

## Contenedor rootless

En este ejemplo vamos a ejecutar nuestro primer contenedor rootless, es decir en esta ocasión el usuario que va a crear el contenedor es un usuario sin privilegio (no usaremos la instrucción `sudo`):

```
$ podman run quay.io/josedom24/hola-mundo:latest
Trying to pull quay.io/josedom24/hola-mundo:latest...
Getting image source signatures
Copying blob a0bed814693a skipped: already exists  
Copying blob c9b9b8ad6639 done   | 
Copying config 3ab8e3c3fd done   | 
Writing manifest to image destination
Hola Mundo!!!

1. Este contenedor se ha iniciado.
2. Ha mostrado el mensaje.
3. Y finalmente se ha parado.
```

En este caso:

1. Hemos ejecutado la instrucción `podman run` con un usuario sin privilegios.
2. Al indicar la imagen hemos usado su nombre completo, indicando el nombre del registro, el nombre del usuario, el nombre de la imagen, y el nombre de la etiqueta (si no se indica el nombre de la etiqueta se escogerá por defecto `latest`).
3. Se crea el contenedor que ejecuta un comando por defecto que muestra el mensaje que hemos leído y se detiene.

Cada usuario gestiona sus imágenes y sus contenedores, por lo tanto el usuario sin privilegio no ve las imágenes ni los contenedores del `root`:

```
$ podman images
REPOSITORY                     TAG          IMAGE ID      CREATED         SIZE
quay.io/josedom24/hola-mundo   latest       3ab8e3c3fd4e  15 minutes ago  7.68 MB

$ podman ps -a
CONTAINER ID  IMAGE                                COMMAND           CREATED        STATUS                    PORTS       NAMES
1e2b7e2a7494  quay.io/josedom24/hola-mundo:latest  ash /opt/hola.sh  3 minutes ago  Exited (0) 8 minutes ago              gracious_dubinsky
```

## Creación de contenedores sin ejecutarlos

De manera habitual vamos a usar `podman run` para crear y ejecutar un contenedor. Podríamos también crear un contenedor que no se ejecute y posteriormente dar la orden de ejecución. Vamos a observar que la creación de este segundo contenedor será mucho más rápida ya que tenemos la imagen descargada en nuestro registro local. Para crear un contenedor y no iniciar su ejecución utilizaremos el comando `podman create`:

```
$ sudo podman create hello-world
7eb70aea838788181a00ad14e3e447ef231dcff680fc37f7b0025e551080fbf8
```

Podemos ver que el contenedor está creado pero no en ejecución:


```
$ sudo podman ps -a
CONTAINER ID  IMAGE                            COMMAND               CREATED         STATUS                     PORTS       NAMES
7eb70aea8387  quay.io/podman/hello:latest      /usr/local/bin/po...  12 seconds ago  Created                                pensive_gagarin

```

Podemos iniciar la ejecución de este contenedor usando `podman start -a`. La opción `-a` nos permite conectar a la salida estándar del contenedor y poder ver en nuestro terminal la salida.

```
$ sudo podman start -a pensive_gagarin
!... Hello Podman World ...!
...
```
