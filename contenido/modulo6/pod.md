# Ejecución de Pods con Systemd y Quadlet

## Ejecución de ficheros YAML de Kuberenetes con Systemd y Quadlet

En este ejemplo vamos a crear una unidad de systemd que cree un Pod a partir de un YAML de Kubernetes ejecutando internamente `podman kube play`. Para ello partimos de la siguiente definición de recurso en YAML.

El contenido del fichero `web-pod.yaml` es el siguiente:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-04-05T06:53:21Z"
  labels:
    app: webserver-pod
  name: webserver-pod
spec:
  containers:
  - args:
    - /usr/bin/run-httpd
    image: quay.io/fedora/httpd-24:2.4
    name: webserver
    ports:
    - containerPort: 8080
      hostPort: 8090
    securityContext:
      runAsNonRoot: true
```

A continuación, creamos una plantilla de unidad de Systemd, en el directorio `/etc/containers/systemd`, llamada `web.kube`:

```
[Unit]
Description=Servidor web a partir de definición YAML de Kubernetes

[Kube]
Yaml=/opt/web-pod.yaml

[Install]
WantedBy=multi-user.target default.target
```

Podemos iniciar la unidad de systemd y comprobar los recursos creados:

```
# systemctl daemon-reload

# systemctl start web

# podman pod ps
POD ID        NAME           STATUS      CREATED        INFRA ID      # OF CONTAINERS
26d86d7df4ab  webserver-pod  Running     8 seconds ago  4511359a3f29  2

# podman ps --pod
CONTAINER ID  IMAGE                                    COMMAND               CREATED         STATUS         PORTS                 NAMES                    POD ID        PODNAME
6dacfe911f42  quay.io/fedora/httpd-24:2.4              /usr/bin/run-http...  17 seconds ago  Up 15 seconds  0.0.0.0:8090->8080/tcp  webserver-pod-webserver  1f94f1bf3daf  webserver-pod
```

## Ejecución de Pods con Systemd y Quadlet (Podman 5)

En este ejemplo, vamos a gestionar un Pod Rootful con un contenedor ofreciendo un servidor web de prueba.

Para ello vamos a crear dos plantillas de unidad de Systemd, en el directorio `/etc/containers/systemd`.

En la plantilla `pod_web.pod` definimos el Pod indicando el nombre y el puerto que va a mapear:

```
[Pod]
PodName=pod-servidorweb
PublishPort=8889:80
```
Además podríamos indicar la red a la que está conectada el Pod usando el parámetro `Network` y el volumen compartido para los contenedores con el parámetro `Volume`.

A continuación definimos el contenedor en la plantilla `webserver.container`:

```
[Unit]
Description=Un contenedor webserver en un Pod

[Container]
Image=quay.io/libpod/banner
ContainerName=contenedor_webserver
Pod=pod_web.pod

[Service]
Restart=always
TimeoutStartSec=900

[Install]
WantedBy=multi-user.target default.target
```

Y podemos iniciar el servicio, ejecutando:

```
$ sudo systemctl start webserver
```

Comprobamos los recursos que hemos creado:

```
$ sudo podman pod ps --ctr-names
POD ID        NAME             STATUS      CREATED         INFRA ID      NAMES
a05c8648fb84  pod-servidorweb  Running     57 seconds ago  e4add95514e8  a05c8648fb84-infra,contenedor_webserver
```

Y finalmente, comprobamos que funciona:

```
$ curl http://localhost:8889
   ___          __              
  / _ \___  ___/ /_ _  ___ ____ 
 / ___/ _ \/ _  /  ' \/ _ `/ _ \
/_/   \___/\_,_/_/_/_/\_,_/_//_/
```
