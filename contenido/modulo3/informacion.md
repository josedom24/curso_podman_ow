# Obteniendo información de las imágenes

## Búsqueda de imágenes

Si queremos buscar imágenes de algún registro remoto desde la línea de comandos, podemos usar la instrucción `podman search`. Podemos indicar el nombre del registro o solo el nombre de la imagen:

```
$ podman search registry.access.redhat.com/httpd
$ podman search httpd
```

## Inspeccionar imágenes

Es posible obtener información detallada sobre una imagen. Para ello usaremos la instrucción `podmanimage inspect` o de forma abreviada:

```
$ podman inspect docker.io/nginx:stable
```

La información más destacable que podemos ver:

* El id y el checksum de la imagen.
* Los puertos que se exponen al crear un contenedor.
* La arquitectura y el sistema operativo de la imagen.
* El tamaño de la imagen.
* Las variables de entorno definidas en la imagen.
* El comando que ejecuta el contenedor que se cree a partir de la imagen.
* Las capas.
* Y muchas más cosas...

Podemos usar filtros, como en el caso de los contenedores. Por ejemplo, para mostrar el identificador de la imagen, ejecutamos

```
$ podman inspect --format='{{.Id}}' docker.io/nginx:stable
```

Consultar los puertos que expone el contenedor que creemos a a partir de esta imagen:

```
$ podman inspect --format='{{range $port,$key := .Config.ExposedPorts}}{{$port}}{{end}}' docker.io/nginx:stable
```

Consultar el sistema operativo y la arquitectura:

```
$ podman inspect --format='{{.Os}} {{.Architecture}}' docker.io/nginx:stable
```

Consultar las variables de entorno definidas en la imagen:

```
$ podman inspect --format='{{range .Config.Env}}{{println .}}{{end}}' docker.io/nginx:stable
```

Y por último, para consultar los identificadores de las capas que forman la imagen:

```
$ podman inspect --format='{{range .RootFS.Layers}}{{println .}}{{end}}' docker.io/nginx:stable
```

Para visualizar los identificadores de las capas que forman parte de la imagen, podemos ejecutar:

```
$ podman image tree docker.io/nginx:stable
Image ID: 7f0fd59e0094
Tags:     [docker.io/library/nginx:stable]
Size:     146.7MB
Image Layers
├── ID: 3c8879ab2cf2 Size: 84.17MB
├── ID: 40e0cedc54cd Size: 62.52MB
├── ID: c84b6e6a1bc2 Size: 3.584kB
├── ID: e5f52d8379ae Size: 4.608kB
├── ID: b5fa8ef12cb9 Size: 3.584kB
└── ID: 3688c0e283b4 Size: 7.168kB Top Layer of: [docker.io/library/nginx:stable]
```

Para comparar los sistemas de archivos de las imágenes podemos usar el comando `podman image diff`. Como resultado aparecerá la lista de archivos y una letra: `C` si ha cambiado, `A` si se ha añadido o `D` si se ha borrado. Si indicamos una sola imagen, esta se comparará con su capa inferior, si damos el nombre de dos imágenes se compararán entre ellas. Por ejemplo:

```
$ podman image diff docker.io/nginx:stable
$ podman image diff docker.io/nginx:stable docker.io/nginx:latest
```

## Montaje de imágenes

Podman nos permite montar el sistema de archivos raíz de una imagen en modo sólo lectura sin crear un contenedor a partir de ella. La imagen montada estará disponible inmediatamente en el sistema anfitrión, permitiéndole examinar su contenido. Para ello usaremos la instrucción `podman image mount` para montar la imagen, y `podman image unmount` para desmontarla.

Por ejemplo, podemos montar una imagen descargada por el usuario `root`:

```
$ sudo podman image mount docker.io/library/nginx
/var/lib/containers/storage/overlay/dd6407eb3a3a9b1d1a18c5d41244b1f989d5cd69a92cc39cd5bab1149d5d8f31/merged

$ sudo podman image mount docker.io/library/nginx
/var/lib/containers/storage/overlay/dd6407eb3a3a9b1d1a18c5d41244b1f989d5cd69a92cc39cd5bab1149d5d8f31/merged

$ sudo podman image unmount docker.io/library/nginx
92b11f67642b62bbb98e7e49169c346b30e20cd3c1c034d31087e46924b9312e
```

Si queremos montar una imagen descargada por un usuario sin privilegios:

```
 podman image mount docker.io/library/nginx
Error: cannot run command "podman image mount" in rootless mode, must execute `podman unshare` first
```

La razón de este error es que el modo rootless no permite montar imágenes. Necesitamos acceder al namespace de usuario sin privilegio, para ello tenemos que usar la instrucción `podman unshare`, que permitirá el acceso a los nombres de espació de usuario y de montaje hasta que indiquemos la instrucción `exit`.

```
$ podman unshare
# podman image mount docker.io/library/nginx
/home/fedora/.local/share/containers/storage/overlay/dd6407eb3a3a9b1d1a18c5d41244b1f989d5cd69a92cc39cd5bab1149d5d8f31/merged
# ls /home/fedora/.local/share/containers/storage/overlay/dd6407eb3a3a9b1d1a18c5d41244b1f989d5cd69a92cc39cd5bab1149d5d8f31/merged
bin  boot  dev  docker-entrypoint.d  docker-entrypoint.sh  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
# podman image unmount docker.io/library/nginx
92b11f67642b62bbb98e7e49169c346b30e20cd3c1c034d31087e46924b9312e
# exit
```

