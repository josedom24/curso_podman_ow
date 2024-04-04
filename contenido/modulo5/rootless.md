
```
mkdir web
echo "<?php phpinfo();?>"> web/index.php
```

En el fichero `web/server_block.conf`:

```
server {
  listen 0.0.0.0:8080;
  server_name myapp.com;

  root /app;

  location / {
    try_files $uri $uri/index.php;
  }

  location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi.conf;
  }
}
```

Creamos:

```
podman pod create --name nginx_php -v ${PWD}/web:/app:z -p 8090:8080
podman run --pod nginx_php --name nginx -d -v ${PWD}/web/server_block.conf:/opt/bitnami/nginx/conf/server_blocks/yourapp.conf:Z docker.io/bitnami/nginx
podman run --pod nginx_php --name fpm_php -d docker.io/bitnami/php-fpm
```

```
podman pod ps
POD ID        NAME        STATUS      CREATED        INFRA ID      # OF CONTAINERS
56dafee5d502  nginx_php   Running     5 minutes ago  d43308bffcf8  3

$ podman ps --pod
CONTAINER ID  IMAGE                                    COMMAND               CREATED        STATUS        PORTS                   NAMES               POD ID        PODNAME
d43308bffcf8  localhost/podman-pause:4.9.4-1711445992                        5 minutes ago  Up 5 minutes  0.0.0.0:8090->8080/tcp  56dafee5d502-infra  56dafee5d502  nginx_php
5e33c6af13d1  docker.io/bitnami/php-fpm:latest         php-fpm -F --pid ...  5 minutes ago  Up 5 minutes  0.0.0.0:8090->8080/tcp  fpm_php             56dafee5d502  nginx_php
5af6eab919d2  docker.io/bitnami/nginx:latest           /opt/bitnami/scri...  3 minutes ago  Up 3 minutes  0.0.0.0:8090->8080/tcp  nginx               56dafee5d502  nginx_php
```

Comprobamos:



