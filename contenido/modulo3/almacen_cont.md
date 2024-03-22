# Almacenamiento de contenedores

Cuando creamos un contenedor a partir de una imagen ocurren la siguientes cosas:

* Cuando se crea un nuevo contenedor desde una imagen, su sistema de archivos será un sistema de archivos de unión cuyas capas inferiores (`lowerdir`) serán las capas de la imagen y la capa superior (`upperdir`) será una nueva capa que denominamos **capa del contenedor**, de lectura y escritura y donde se escribirán todas las diferencias del sistema de archivos del contenedor...
* Los **contenedores son efímeros**, por que cuando lo borramos, se borra la capa del contenedor, por lo que se pierde todos sus datos.
* Por lo tanto cuando creamos un contenedor, el almacenamiento en disco es muy pequeño, ya que las capas de la imagen desde las que se ha creado se comparten con el contenedor y la capa del contenedor en un primer momento tiene muy pocos ficheros.
* Si tenemos un contenedor creado a partir de una imagen, **esta imagen no se puede borrar** ya que sus capas forman parte del sistema de archivos del contenedor en ejecución.

![container1](img/container1.png)

## Creación de un contenedor

Vamos a crear un contenedor, y veremos que se crea la **capa del contenedor** de lectura y escritura, que será la capa superior en el sistema de archivo de unión. Esta nueva capa  capa contendrá sólo las diferencias entre las inferiores. Para ello, ejecutamos:
 
```bash
$ sudo podman run -d --name contenedor1 quay.io/centos7/httpd-24-centos7
579635db3532e954d07927bdc32bd435bd082f03a0696ded072f04d762a18775
```

Y vamos a crear un fichero en el nuevo contenedor:

```bash
$ sudo podman exec contenedor1 bash -c "echo 'Ejemplo Podman' > /tmp/tmpfile.txt"
```

En el directorio de almacenamiento (`/var/lib/containers/storage/` en los contenedores rootful) tenemos un directorio llamado `overlay-containers` donde encontramos la información de almacenamiento de los contenedores que hemos creado:

```bash
$ ls overlay-containers
579635db3532e954d07927bdc32bd435bd082f03a0696ded072f04d762a18775  containers.json  containers.lock
```

El directorio que tiene como nombre el identificador del contenedor que hemos creado tiene información del contenedor, y en el fichero `containers.json` tenemos un índice de los contenedores que hemos creado:

```bash
$ cat overlay-containers/containers.json | jq
[
  {
    "id": "579635db3532e954d07927bdc32bd435bd082f03a0696ded072f04d762a18775",
    "names": [
      "contenedor1"
    ],
    "image": "d7af31210b288164c319bae740ca1281528390a3c5cee657e95f243670b49e6a",
    "layer": "67b0c66296f7957a0d82c8e48442ee0d7e3b3386dadde46cd8dadf3c90d40000",
    "metadata": "{\"image-name\":\"quay.io/centos7/httpd-24-centos7:latest\",\"image-id\":\"d7af31210b288164c319bae740ca1281528390a3c5cee657e95f243670b49e6a\",\"name\":\"contenedor1\",\"created-at\":1711141194}",
    "created": "2024-03-22T20:59:54.545266929Z",
    "flags": {
      "MountLabel": "system_u:object_r:container_file_t:s0:c14,c100",
      "ProcessLabel": "system_u:system_r:container_t:s0:c14,c100"
    }
  }
]
```

En el campo `layer` tenemos el identificador de la **capa del contenedor**, donde se irán escribiendo las diferencias de los ficheros del contenedor respectos a las capas inferiores correspondientes a la imagen. Por otro lado, hay que indicar que esta capa es temporal, existe mientras exista el contenedor, es por lo que decimos que los **contenedores son efímeros**.


Por lo tanto podemos ver los ficheros que hemos escrito en esta capa:

```bash
$ ls overlay/67b0c66296f7957a0d82c8e48442ee0d7e3b3386dadde46cd8dadf3c90d40000/diff/tmp
tmpfile.txt
```

Finalmente vamos a fijarnos en el directorio donde se guarda información del contenedor. Este directorio se llama `userdata`:

```
# cd overlay-containers/ae697efd8d29d8d75988390c19cf787ed8057bacfb1cea82a93a2c36756f88ee/userdata/
[root@podman userdata]# ls
artifacts  attach  config.json  ctl  secrets  shm  winsz
```

Este directorio contiene varios archivos que se montan directamente en el contenedor para personalizarlo.


## Creación del sistema de ficheros superpuesto

![container2](img/container2.png)

podman inspect --format='{{range $key,$dir := .GraphDriver.Data}}{{$key}} = {{$dir}}\n{{end}}'  contenedor1


mount | grep overlay

## Ejemplo de almacenamiento de contenedores


