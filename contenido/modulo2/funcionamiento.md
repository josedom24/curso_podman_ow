# ¿Cómo funcionan los contenedores rootless?

Ante de estudiar el funcionamiento de los contenedores rootless, vamos a ver cómo funcionan los contenedores rootful.

## Ejecución de contenedores rootful

Como hemos indicado un contenedor rootful es un contenedor ejecutado por root en el host. Pero, ¿Qué usuario ejecuta los procesos dentro del contenedor?. La respuesta a esta pregunta nos ofrece dos posibilidades:

### Ejecución de contenedores rootful, con procesos en el contenedor ejecutándose como root

Veamos un ejemplo:

```
# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

# podman run -d --rm --name contenedor1 alpine sleep 1000
c7cbb189f807fba22a77b465586f9d1e048f199ebca6b674fe8d82fc9be82f5b

# podman exec contenedor1 id
uid=0(root) gid=0(root)

# ps -ef | grep sleep
root       22702   22700  0 08:59 ?        00:00:00 sleep 1000

# podman top contenedor1 huser user 
HUSER       USER
root        root
```

1. En primer lugar vemos que el usuario que va a crear el contenedor es `root`.
2. Comprobamos que el usuario que está ejecutando los procesos dentro del contenedor es `root`.
3. Comprobamos que en el host, el proceso lo ejecuta el usuario `root`.
4. Por último, vemos con la instrucción `podman top`, el usuario correspondiente al host (`HUSER`) que es `root`, y el usuario dentro del contenedor (`USER`) que en este caso, también es `root`.

Podemos concluir: cuando ejecutamos un contenedor como `root`, con el proceso del contenedor ejecutándose como `root`, el usuario real visible en el host que ejecuta el proceso es `root` con UID 0.

### Ejecución de contenedores rootful, con procesos en el contenedor ejecutándose con usuarios no privilegiados

Al ejecutar un contenedor podemos usar el parámetro `--user` o `-u` para indicar el usuario que se utilizará dentro del contenedor para ejecutar los procesos. El usuario también puede venir indicado en la imagen que estemos utilizando. Veamos un ejemplo:

```
# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

# podman run -d --rm -u sync --name contenedor1 alpine sleep 1000
6bf3ec46d7f391e3240cbce4e159adfea7d780435f3d324023a8b0875b83fa56

# podman exec contenedor1 id
uid=5(sync) gid=0(root)

#  ps -ef | grep sleep
sync       23135   23133  0 09:37 ?        00:00:00 sleep 1000

# podman top contenedor1 huser user 
HUSER       USER
sync        sync
```
En este caso:

1. En primer lugar vemos que el usuario que va a crear el contenedor es `root`.
2. Comprobamos que el usuario que está ejecutando los procesos dentro del contenedor es `sync`.
3. Comprobamos que en el host, el proceso lo ejecuta el usuario `sync`.
4. Por último, vemos que el usuario correspondiente al host (`HUSER`) es `sync`, y el usuario dentro del contenedor (`USER`) también es `sync`.

Podemos concluir: cuando ejecutamos un contenedor como `root`, con el proceso del contenedor ejecutándose un usuario sin privilegios, el usuario real visible en el host que ejecuta el proceso es el usuario sin privilegio, con el UID del usuario que ejecuta el proceso dentro del contenedor.

## Ejecución de contenedores rootless

Veamos ahora cómo se comportan las asignación de usuarios cuando ejecutamos contenedores rootless. En nuestro caso, vamos a a usar el usuario `usuario` con UID 1000 para crear los contenedores. De la misma forma que anteriormente, podemos ejecutar los procesos dentro del contenedor con diferente usuarios:

### Ejecución de contenedores rootless, con procesos en el contenedor ejecutándose como root

Veamos un ejemplo:

```
$ id
uid=1000(usuario) gid=1000(usuario) groups=1000(usuario),4(adm),10(wheel),190(systemd-journal) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

$ podman run -d --rm --name contenedor1 alpine sleep 1000
f3961860f97280adf64c44a8b42dd39588712d3935469bf97d3ae7d71b8ffa97

$ podman exec contenedor1 id
uid=0(root) gid=0(root)

$ ps -ef | grep sleep
usuario     23234   23232  0 09:47 ?        00:00:00 sleep 1000

$ podman top contenedor1 huser user
HUSER       USER
1000        root
```

1. En primer lugar vemos que el usuario que va a crear el contenedor es `usuario`.
2. Comprobamos que el usuario que está ejecutando los procesos dentro del contenedor es `root`.
3. Comprobamos que en el host, el proceso lo ejecuta el usuario `usuario`.
4. Por último, vemos que el usuario correspondiente al host (`HUSER`) es `usuario` (UID 1000), y el usuario dentro del contenedor (`USER`) es `root`. **Hay una correspondencia entre nuestro usuario sin privilegios en el host y el usuario `root` dentro del contenedor**.

Podemos concluir: cuando ejecutamos un contenedor con un usuario sin privilegios, con el proceso del contenedor ejecutándose con `root`, el usuario real visible en el host que ejecuta el proceso es el usuario sin privilegio con su UID.

### Ejecución de contenedores rootful, con procesos en el contenedor ejecutándose con usuarios no privilegiados

En el caso de los contenedores rootless, también podemos indicar el usuario que ejecutara los procesos dentro del contenedor con el parámetro `--user` o `-u`, o utilizando una imagen donde venga definido. Veamos un ejemplo:

```
$ id
uid=1000(usuario) gid=1000(usuario) groups=1000(usuario),4(adm),10(wheel),190(systemd-journal) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

$ podman run -d --rm -u sync --name contenedor1 alpine sleep 1000
96d64bf75b7998de86624dd699f450f83670b4a798e775585edc8c2607de94ca

$ podman exec contenedor1 id
uid=5(sync) gid=0(root)

$ ps -ef | grep sleep
524292     23377   23375  0 09:57 ?        00:00:00 sleep 1000

$ podman top contenedor1 huser user
HUSER       USER
524292      sync
```

En este caso:

1. En primer lugar vemos que el usuario que va a crear el contenedor es `usuario`.
2. Comprobamos que el usuario que está ejecutando los procesos dentro del contenedor es `sync`.
3. Comprobamos que en el host, el proceso lo ejecuta un usuario con UID 524292.
4. Por último, vemos que el usuario correspondiente al host (`HUSER`)es un usuario con UID 524292, y el usuario dentro del contenedor (`USER`) es `sync`. **Hay una correspondencia entre un usuario sin privelegios en el host y el usuario `sync` dentro del contenedor**.

Podemos concluir: cuando ejecutamos un contenedor con un usuario sin privilegios, con el proceso del contenedor ejecutándose con un usuario sin privilegio, el usuario real visible en el host que ejecuta el proceso es otro usuario con privilegio con un UID propio.

En resumen:

![resumen](img/rootless.png)

## Espacio de nombres de usuario

Los espacios de nombres (**namespaces**) son un mecanismo que el kernel utiliza para aislar y restringir recursos del sistema operativo, como procesos, redes, sistemas de archivos, entre otros. Los namespaces permiten crear entornos de ejecución independientes

Cuando ejecutamos contenedores rootless, hemos visto que podemos ejecutar los procesos dentro del contenedor con otros usuarios. Además dentro de la imagen que estamos usando para crear el contenedor pueden estar definidos varios usuarios. Sin embargo, el kernel de Linux impide a un usuario sin privilegios usar más de UID, por ello necesitamos un mecanismo que consiga que nuestro usuario sin privilegio pueda utilizar distintos UID y GID.

Es por todo ello, que se use un espacio de nombre de usuario (**user username**):

* Nos permite **asignar un rango de IDs de usuario y grupo** en un espacio de nombres aislado. Esto significa que los procesos que se ejecutan dentro de ese namespace tienen una visión limitada de los usuarios y grupos del sistema en comparación con el sistema anfitrión. 
* Nos permite establecer una correspondencia entre los ID de usuario del contenedor y los ID de usuario del host.
* En el espacio de nombres de usuario de Podman, hay un nuevo conjunto de IDs de usuario e IDs de grupo, que están separados de los UIDs y GIDs de su host.

Por ejemplo, como vimos en los ejemplos anteriores:

* Cuando creamos un contenedor rootless donde se ejecutan los procesos como `root` (uid = 0), en el host se están ejecutando con el usuario que ha creado el contenedor, en nuestro caso con el usuario `usuario` (uid = 1000).
* Cuando creamos un contenedor rootless donde se ejecutan los procesos con el usuario `sync` (uid = 5), en el host se están ejecutando con un usuario sin privilegios con uid = 524292.

![rootless](img/rootless2.png)

Cada usuario no privilegiado que creemos en nuestro host, tenfra un conjunto de UID y GID que podrá mapear a usuarios y grupos dentro del contenedor:

* En el fichero `/etc/subuid`, por cada usuario tenemos el UID inicial y la cantidad de identificadores que puede mapear. Cada usuario tiene que tener un conjunto de identificadores diferentes.
    ```
    $ cat /etc/subuid
    usuario:524288:65536
    ```

    El usuario `usuario` puede mapear desde el UID 524288 y tiene asignado 65536 identificadores.
* En el fichero `/etc/subgid` está definido, con el mismo formato los identificadores de grupos que puede mapear cad usuario.


Aislamiento de privilegios: Cuando se ejecutan contenedores en modo rootless, el usuario no necesita privilegios de superusuario para iniciar y administrar los contenedores. El user namespace permite asignar un conjunto de IDs de usuario y grupo dentro del contenedor que son diferentes de los IDs en el sistema anfitrión. Esto proporciona una capa adicional de aislamiento de seguridad, ya que los procesos dentro del contenedor no tienen privilegios en el sistema anfitrión.

Seguridad mejorada: Al limitar los privilegios del contenedor a través del user namespace, se reduce el riesgo de que un contenedor comprometido pueda acceder o modificar recursos críticos del sistema anfitrión.