# Ejecución de un contenedor con un servidor de base de datos

En ocasiones es obligatorio el inicializar alguna variable de entorno para que el contenedor pueda ser ejecutado. Si miramos la [documentación](https://hub.docker.com/_/mariadb) en Docker Hub de la imagen `mariadb`, observamos que podemos definir algunas variables de entorno para la creación y configuración del contenedor (por ejemplo: `MARIADB_DATABASE`,`MARIADB_USER`, `MARIADB_PASSWORD`,...). Pero hay una que la tenemos que indicar de forma obligatoria, la contraseña del usuario `root` (`MARIADB_ROOT_PASSWORD`). Podríamos ejecutar el contenedor indicando directamente la variable de entorno:

```
$ podman run -d --name mimariadb -e MARIADB_ROOT_PASSWORD=my-secret-pw docker.io/mariadb:10.5
```

Podemos ver que se ha creado una variable de entorno:

```
$ podman exec -it mimariadb env
...
MARIADB_ROOT_PASSWORD=my-secret-pw
...
```

Y para acceder podemos ejecutar:

```
$ podman exec -it mimariadb bash
root@9c3effd891e3:/# mysql -u root -p"$MARIADB_ROOT_PASSWORD" 
...

MariaDB [(none)]> 
```
Otra forma de hacerlo sería:

```
$ podman exec -it mimariadb mysql -u root -p -h 127.0.0.1
Enter password: 
...
MariaDB [(none)]> 
```
## Uso de Secrets para la configuración del contenedor

Como hemos indicado anteriormente, al ser la contraseña una información sensible, vamos a guardar dicho valor en un Secret y posteriormente creamos el contenedor:

```
$ echo "my-secret-pw" | podman secret create pass_root -
$ podman run -d --name mimariadb2 --secret pass_root,type=env,target=MARIADB_ROOT_PASSWORD docker.io/mariadb:10.5
```

Podemos ver que se ha creado una variable de entorno:

```
$ podman exec -it mimariadb2 env
...
MARIADB_ROOT_PASSWORD=my-secret-pw
...
```

Y para acceder podemos ejecutar:

```
$ podman exec -it mimariadb2 bash
root@9c3effd891e3:/# mysql -u root -p"$MARIADB_ROOT_PASSWORD" 
...

MariaDB [(none)]> 
```



## Accediendo a servidor de base de datos desde el exterior

En el ejemplo anterior hemos accedido a la base de datos de dos formas: 

1. Ejecutado un comando `bash` para acceder al contenedor y desde dentro hemos utilizado el cliente de MariaDB para acceder a la base de datos.
2. Ejecutando directamente en el contenedor el cliente de MariaDB.

En esta ocasión vamos a mapear los puertos para acceder desde el exterior a la base de datos: vamos a mapear el puerto 3306/tcp del Host Docker con el puerto 3306/tcp del contenedor:

``` 
$ podman run -d -p 3306:3306 --name mimariadb3 -e MARIADB_ROOT_PASSWORD=my-secret-pw docker.io/mariadb:10.5
```

Comprobamos que los puertos se han mapeado y que el contenedor está ejecutándose:

```
$ podman ps
CONTAINER ID  IMAGE                           COMMAND     CREATED        STATUS        PORTS                   NAMES
ee15342ca308  docker.io/library/mariadb:10.5  mysqld      3 seconds ago  Up 3 seconds  0.0.0.0:3306->3306/tcp  mimariadb3
```

Ahora desde nuestro equipo, donde hemos instalado un cliente de MariaDB (`sudo apt install mariadb-client` en Debian/Ubuntu o `sudo dnf install community-mysql.x86_64` en Fedora), nos conectamos al host:

```
$ mysql -u root -p -h 127.0.0.1
Enter password: 
...
mysql> 
```
