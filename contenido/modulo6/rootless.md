# Ejecución de contenedores rootless con Systemd y Quadlet

Si queremos ejecutar recursos Podman como servicios con systemd, tenemos que guardar las plantillas de unidades de systemd en el directorio `$HOME/.config/containers/systemd`.

Es posible que ese directorio lo tengamos que crear:

```
$ mkdir -p ~/.config/containers/systemd
```


Por ejemplo, podemos guardar la plantilla `rootless.container` en el directorio indicado con este contenido:

```
[Unit]
Description=Un contenedor rootless con un servidor web

[Container]
Image=quay.io/libpod/banner
ContainerName=contenedor_rootless
PublishPort=8888:80

[Service]
Restart=always
TimeoutStartSec=900

[Install]
WantedBy=multi-user.target default.target
```

Y a continuación reiniciar systemd en el espacio de usuario y arrancar la unidad de sistema en el espacio del usuario:

```
$ systemctl --user daemon-reload
$ systemctl --user start rootless
```

Ahora podemos acceder a comprobar el acceso a la aplicación:

```
$ curl http://localhost:8888
   ___          __              
  / _ \___  ___/ /_ _  ___ ____ 
 / ___/ _ \/ _  /  ' \/ _ `/ _ \
/_/   \___/\_,_/_/_/_/\_,_/_//_/
```

