# Gestión de redes definidas por el usuario

Además de poder usar la red **bridge** por defecto, podemos crear nuevas redes de este tipo, a las que llamamos **redes bridge definidas por el usuario**. Como hemos visto en la introducción este tipo de redes nos proporcionan un mecanismo **DNS** que nos permite el acceso entre contenedores usando su nombre. Este tipo de redes serán las deseadas en entornos de producción.

## Gestión de redes bridge definidas por el usuario

Para crear una red bridge definida por el usuario, ejecutamos la siguiente instrucción: 

```
$ sudo podman network create -d bridge red1
```

La opción `-d bridge` es optativa, si no se se indica el tipo, por defecto se creará una red de tipo bridge. Podemos visualizar las redes que tenemos creadas ejecutando:

```
$ sudo podman network ls
NETWORK ID    NAME        DRIVER
2f259bab93aa  podman      bridge
dd36a02e4956  red1        bridge
```
Para obtener información de la red, podemos ejecutar:

```
$ sudo podman network inspect red1
...
    "subnets": [
               {
                    "subnet": "10.89.0.0/24",
                    "gateway": "10.89.0.1"
               }
          ],
...
```
Al crear la red no hemos indicado el direccionamiento, por lo que se ha asignado uno por defecto. Además en el host se creará un nuevo bridge donde se conectarán los contenedores que estén conectados a esta red. El Linux Bridge se crea cuando le conectemos un contenedor.

Por último, para borrar una red podemos ejecutar:

```
$ sudo podman network rm red1
```

Para borrar todas las redes que no están siendo usadas al menos por un contenedor, ejecutamos:

```
$ sudo podman network prune
WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N]
```

Teniendo en cuenta que no puedo borrar una red que tenga contenedores que la estén usando, deberé primero borrar los contenedores o desconectarlos de la red.

## Creación avanzada de redes bridge definidas por el usuario

En la creación de una red de tipo bridge podemos configurar algunos parámetros, por ejemplo el direccionamiento y la puerta enlace. Veamos un ejemplo:

```
$ sudo podman network create --subnet 192.168.0.0/24 --gateway 192.168.0.100 red2
```

Podemos comprobar que el direccionamiento lo hemos configurado:

```
$ sudo podman network red2
...
"subnets": [
               {
                    "subnet": "192.168.0.0/24",
                    "gateway": "192.168.0.100"
               }
          ],
...
```

