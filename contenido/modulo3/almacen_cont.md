# Almacenamiento de contenedores

## ¿Qué ocurre cuando creamos un contenedor?

![ ](img/layers.png)

* Cuando se crea un nuevo contenedor desde una imagen, su sistema de archivos será la unión de todas las capas de la imagen. 
* Las capas de la imagen son únicamente de lectura, por lo que se añade una nueva capa de lectura-escritura. 
* Todos los cambios efectuados al contenedor específico son almacenados en esa capa.
* Esta capa se suele llamar **Capa del Contenedor**.
* Los contenedores son efímeros, por que cuando lo borramos, se borra la capa del contenedor, por lo que se pierde todos sus datos.
* Por lo tanto cuando creamos un contenedor, el almacenamiento en disco es muy pequeño, ya que las capas de la imagen desde las que se ha creado se comparten con el contenedor y la capa del contenedor en un primer momento tiene muy pocos ficheros.
* Si tenemos un contenedor creado a partir de una imagen, **esta imagen no se puede borrar** ya que sus capas forman parte del sistema de archivos del contenedor en ejecución.

![ ](img/layers2.png)



## Directorio overlay-container

Finalmente, vamos a crear un contenedor, y veremos que se crea la capa del contenedor de lectura y escritura. Esta nueva capa  capa contendrá sólo las diferencias entre las inferiores. Para ello, ejecutamos:
 
```bash
$ sudo podman run -d quay.io/centos7/httpd-24-centos7
ae697efd8d29d8d75988390c19cf787ed8057bacfb1cea82a93a2c36756f88ee
```

Y vamos a crear un fichero en el nuevo contenedor:

```bash
$ sudo podman exec ae697efd8d29d8d75988390c19cf787ed8057bacfb1cea82a93a2c36756f88ee bash -c "echo 'Ejemplo Podman' > /tmp/tmpfile.txt"
```

En el directorio de almacenamiento tenemos un directorio llamado `overlay-containers` donde encontramos la información de almaceanmiento de los contenedores que hemos creado:

```
cd overlay-containers/
# ls
ae697efd8d29d8d75988390c19cf787ed8057bacfb1cea82a93a2c36756f88ee  containers.json  containers.lock
```

El directorio que tiene como nombre el identificador del contenedor que hemos creado tiene información del contenedor, y en el fichero `containers.json` tenemos un índice de los contenedores que hemos creado:

```
# cat containers.json | jq
[
  {
    "id": "ae697efd8d29d8d75988390c19cf787ed8057bacfb1cea82a93a2c36756f88ee",
    "names": [
      "stoic_cerf"
    ],
    "image": "d7af31210b288164c319bae740ca1281528390a3c5cee657e95f243670b49e6a",
    "layer": "206efdc36850d2f4d47776ef079828b712e07be796209122ba988df4a3e1362b",
    "metadata": "{\"image-name\":\"quay.io/centos7/httpd-24-centos7:latest\",\"image-id\":\"d7af31210b288164c319bae740ca1281528390a3c5cee657e95f243670b49e6a\",\"name\":\"stoic_cerf\",\"created-at\":1711009710}",
    "created": "2024-03-21T08:28:30.380877163Z",
    "flags": {
      "MountLabel": "system_u:object_r:container_file_t:s0:c186,c931",
      "ProcessLabel": "system_u:system_r:container_t:s0:c186,c931"
    }
  }
]
```
En el campo `layer` tenemos el identificador de la capa del contenedor, donde se irán escribiendo las diferencias de los ficheros del contenedor respectos a las capas inferiores correspondientes a la imagen. Por otro lado, hay que indicar que esta capa es temporal, existe mientras exista el contenedor, es por lo que decimos que los contenedores son efímeros.


Por lo tanto podemos ver los ficheros que hemos escrito en esta capa:

```
# cd overlay/206efdc36850d2f4d47776ef079828b712e07be796209122ba988df4a3e1362b/diff/tmp/
# ls
tmpfile.txt
```

Finalmente vamos a fijarnos en el directorio donde se guarda información del contenedor. Este directorio se llama `userdata`:

```
# cd overlay-containers/ae697efd8d29d8d75988390c19cf787ed8057bacfb1cea82a93a2c36756f88ee/userdata/
[root@podman userdata]# ls
artifacts  attach  config.json  ctl  secrets  shm  winsz
```

Este directorio contiene varios archivos que se montan directamente en el contenedor para personalizarlo.


podman inspect --format='{{range $key,$dir := .GraphDriver.Data}}{{$key}} = {{$dir}}\n{{end}}'  stoic_cerf


mount | grep overlay
