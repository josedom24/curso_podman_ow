## Visualizando las imágenes de nuestro registro local

Con el comando `docker images` (también se puede usar `docker image ls`, `docker image list`) podemos visualizar las imágenes que ya tenemos descargadas en nuestro registro local:

```bash
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
ubuntu        latest    e34e831650c1   11 days ago    77.9MB
hello-world   latest    d2c94e258dcb   8 months ago   13.3kB
```


--------


Otra cosa que me he encontrado, cuando no sabe de qué registro bajar la imagen, te pregunta

```
sudo podman run -d --name my-apache-app -p 8080:80 httpd:2.4
? Please select an image: 
  ▸ registry.fedoraproject.org/httpd:2.4
    registry.access.redhat.com/httpd:2.4
    docker.io/library/httpd:2.4
    quay.io/httpd:2.4
```

```