# Ejecución de contenedores con Systemd y Quadlet

En este ejemplo vamos a gestionar un contenedor rootful que ofrece un servidor web nginx, por lo tanto vamos a escribir la siguiente plantilla de unidad Systemd, en el directorio `/etc/containers/systemd`. El nombre de la plantilla es `nginx.container` y tiene el siguiente contenido:

```
[Unit]
Description=Un contenedor con el servidor web nginx

[Container]
Image=docker.io/nginx
ContainerName=contenedor_nginx
PublishPort=8888:80

[Service]
Restart=always
TimeoutStartSec=900

[Install]
WantedBy=multi-user.target default.target
```

Como vemos el formato de la plantilla es similar al formato de una unidad de Systemd estándar:

* Las secciones y parámetros propias de la definición de una unidad Systemd, por ejemplo `[Unit]`, `[Service]` y `[Install]` se copiaran directamente en el fichero generado. Hemos puesto algunos parámetros comunes de ejemplo, pero podemos indicar los que nos interesen:
    * `Restart=always`: Política de reinicio. en este caso, se intentará reiniciar el servicio siempre que se detenga.
    * `TimeoutStartSec=900`: Tiempo máximo (en segundos) que Systemd esperará a que el servicio se inicie antes de considerarlo como un fallo.
    * `WantedBy=multi-user.target default.target`: El servicio será iniciado automáticamente durante el arranque del sistema.
* En esta plantilla donde vamos a definir la ejecución de un contenedor tenemos una sección especial que se llama `[Container]`. Dentro de esta sección pondremos distintos parámetros para especificar las características que tendrá el contenedor que vamos a gestionar. Esta sección no aparecerá en la unidad Systemd generada, pero los parámetros indicados se utilizarán para generar la configuración adecuada. Los parámetros más importantes que podemos indicar dentro de la sección `[Container]` son:
    * `Image`: Nos permite indicar la imagen desde la que se crea el contenedor.
    * `PublishPort`: Nos permite mapear puertos. Igual que la opción `-p` de `podman run`.
    * `ContainerName`: Nos permite indicar el nombre del contenedor.
    * `Environment`: Nos permite definir variables de entorno. Igual que la opción `-e` de `podman run`.
    * `Exec`: Nos permite indicar el comando que queremos que se ejecute en el contenedor. 
    * `Network`: Nos permite indicar la red donde se conecta el contenedor. Igual que la opción `--netowrk` de `podman run`.
    * `Pod`: Nos permite añadir el contenedor en un Pod. Igual que la opción `--pod` de `podman run`.
    * `Volume`: Nos permite añadir almacenamiento al contenedor. Igual que la opción `--volume` de `podman run`.
    * `PodmanArgs`: nos permite añadir parámetros extras a la comando `podman run`.
    * Podemos indicar más parámetros que puedes consultar en la [documentación](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html).

Una vez que tenemos definida la plantilla y siempre que la modifiquemos tendremos que reiniciar Systemd:

```
# systemctl daemon-reload
```

Y ya podemos trabajar con la unidad de Systemd que se ha generado:

```
# systemctl status nginx
○ nginx.service - Un contenedor con el servidor web nginx
     Loaded: loaded (/etc/containers/systemd/nginx.container; generated)
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
     Active: inactive (dead)

# systemctl start nginx
# systemctl status nginx
● nginx.service - Un contenedor con el servidor web nginx
     Loaded: loaded (/etc/containers/systemd/nginx.container; generated)
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
     Active: active (running) since Tue 2024-04-09 07:12:43 UTC; 30s ago
   Main PID: 74403 (conmon)
      Tasks: 4 (limit: 1087)
     Memory: 18.9M
        CPU: 864ms
     CGroup: /system.slice/nginx.service
             ├─libpod-payload-9687bac58d5c4d1606aa46af33d1a21c1d1652e0ac9fb3fc0fc86e3495f42869
             │ ├─74405 "nginx: master process nginx -g daemon off;"
             │ ├─74430 "nginx: worker process"
             │ └─74431 "nginx: worker process"
    ...

# podman ps
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS         PORTS                 NAMES
0bacbe10921c  docker.io/library/nginx:latest  nginx -g daemon o...  4 seconds ago   Up 2 seconds   0.0.0.0:8888->80/tcp  contenedor_nginx


# curl http://localhost:8888
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

# systemctl stop nginx
```

Por último si queremos obtener la unidad Systemd generada, podemos ejecutar:

```
# /usr/libexec/podman/quadlet -dryrun 

quadlet-generator[74499]: Loading source unit file /etc/containers/systemd/nginx.container
---nginx.service---
[Unit]
Description=Un contenedor con el servidor web nginx
SourcePath=/etc/containers/systemd/nginx.container
RequiresMountsFor=%t/containers

[X-Container]
Image=docker.io/nginx
ContainerName=contenedor_nginx
PublishPort=8888:80

[Service]
Restart=always
TimeoutStartSec=900
Environment=PODMAN_SYSTEMD_UNIT=%n
KillMode=mixed
ExecStop=/usr/bin/podman rm -v -f -i --cidfile=%t/%N.cid
ExecStopPost=-/usr/bin/podman rm -v -f -i --cidfile=%t/%N.cid
Delegate=yes
Type=notify
NotifyAccess=all
SyslogIdentifier=%N
ExecStart=/usr/bin/podman run --name=contenedor_nginx --cidfile=%t/%N.cid --replace --rm --cgroups=split --sdnotify=conmon -d --publish 8888:80 docker.io/nginx

[Install]
# Start by default on boot
WantedBy=multi-user.target default.target
```
