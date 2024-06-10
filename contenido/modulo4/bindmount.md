# Trabajando con bind mount

## Creación de contenedores con bind mount con la opción --volumen

Seguimos en este ejemplo, utilizando contenedores rootful. En este caso, vamos a crear un directorio en el sistema de archivos del host, donde vamos a crear un fichero `index.html`:

```
$ mkdir web
$ cd web
/web$ echo "<h1>Hola</h1>" > index.html
```

Y podemos montar ese directorio en un contenedor, en este caso usamos la opción `-v`:

```
$ sudo podman run -d --name my-apache-app -v /home/usuario/web:/usr/local/apache2/htdocs -p 8080:80 docker.io/httpd:2.4
```

Comprobamos si estamos sirviendo el fichero que tenemos en el directorio que hemos creado:

```
$ curl http://localhost:8080
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
</body></html>
```

Sin embargo comprobamos que nos da un error: el contenedor no puede acceder al directorio que hemos montado.

## Montaje de directorio cuando usamos SELinux

El ejemplo anterior hubiera funcionado sin problemas, si en nuestro host no tuviéramos activado SELinux. 
En nuestro ejemplo, el error se ha producido porque tenemos activo SELinux y el directorio `/home/usuario/web` no está configurado de forma adecuada para ser accesible desde el contenedor.

Por lo tanto, como hemos visto anteriormente, a la hora de montar el directorio, tendremos que usar la opción `:z` si dicho directorio lo queremos montar en otros contenedores o `:Z` si sólo se va a montar en este contenedor.

Por lo tanto eliminamos el contenedor anterior y lo volvemos a crear de forma adecuada:

```
$ sudo podman rm -f my-apache-app
my-apache-app
$ sudo podman run -d --name my-apache-app -v /home/usuario/web:/usr/local/apache2/htdocs:Z -p 8080:80 docker.io/httpd:2.4
```

Podemos comprobar en la información del contenedor los puntos de montaje que tiene configurado:

```
$ $ sudo podman inspect --format='{{json .Mounts}}' my-apache-app 
[{"Type":"bind","Source":"/home/usuario/web","Destination":"/usr/local/apache2/htdocs","Driver":"","Mode":"","Options":["rbind"],"RW":true,"Propagation":"rprivate"}]
```

Y comprobamos que realmente estamos sirviendo el fichero que tenemos en el directorio que hemos creado.

```
$ curl http://localhost:8080
<h1>Hola</h1>
```

## Creación de contenedores con bind mount con la opción --mount

Cuando configuramos un directorio para ser montado en un contenedor con la opción `:z` o `:Z`, la configuración que se realiza no se deshace aunque eliminemos el contenedor. Ese directorio continúa estando accesible desde los contenedores que creemos y no será necesario volver a usar la opción. Si queremos volver a configurar el directorio con sus permisos de accesos originales, debemos ejecutar:

```
$ sudo restorecon -F -R /home/usuario/web
```

Eliminamos el contenedor y volvemos a crear otro con el directorio montado, ahora usando la opción `--mount`. En este caso, para montar un directorio compartido (similar a usar la opción `:z`), utilizaríamos la opción `relabel=shared`. Si queremos hacer un montaje privado, sólo para el contenedor (similar a la opción `:Z`) usaremos la opción `relabel=private`

```
$ sudo podman rm -f my-apache-app 

$ sudo podman run -d --name my-apache-app --mount type=bind,src=/home/usuario/web,dst=/usr/local/apache2/htdocs,relabel=private -p 8080:80 httpd:2.4


$ curl http://localhost:8080
<h1>Hola</h1>
```

Además, podemos comprobar que al modificar el contenido del fichero se modificará en el contenedor:

```
$ echo "<h1>Adios</h1>" > web/index.html 
$ curl http://localhost:8080
<h1>Adios</h1>
```

Por último, indicar que si nuestro directorio origen no existe y hacemos un bind mount con `-v`, se creará, pero lo que tendremos en el contenedor será un directorio vacío. Sin embargo, si hacemos un bind mount con la opción `--mount` nos dará un error.
