# Ejemplo: Desplegando la aplicación Drupal

[**Drupal**](https://www.drupal.org/) es una aplicación web escrita en PHP que nos permite construir un CMS. En este ejemplo vamos a crear contenedores usando la imagen [`drupal`](https://hub.docker.com/_/drupal) que encontramos en Docker Hub. 

## Instalación de distintas versiones de Drupal

Vamos a crear distintos contenedores usando etiquetas distintas al indicar el nombre de la imagen, posteriormente accederemos a la aplicación y podremos ver la versión instalada.

En primer lugar vamos a instalar la última versión:

```bash
podman run -d -p 8080:80 --name drupal1 docker.io/drupal
```

Si accedemos a la dirección IP de nuestro ordenador, al puerto 8080/tcp, podemos observar que hemos instalado la versión 10.2.4:

![drupal](img/drupal1.png)

A continuación vamos a instalar otra versión de Drupal, la 10.1.8, creamos otro contenedor con otro nombre y mapeamos otro puerto:

```bash
podman run -d -p 8081:80 --name drupal2 docker.io/drupal:10.1.8
```

Si accedemos a la dirección IP de nuestro ordenador, al puerto 8081/tcp, podemos observar la versión que hemos instalado:

![drupal](img/drupal1.png)

Y finalmente vamos a instalar otra versión en otro contenedor:

```bash
podman run -d -p 8081:80 --name drupal2 docker.io/drupal:10.0.11
```

Si accedemos a la dirección IP de nuestro ordenador, al puerto 8082/tcp, podemos observar la versión instalada:

![mediawiki](img/mediawiki1396.png)

**Nota: Puedes observar que la primera imagen que se baja, descarga todas las capas, sin embargo al descargar las otras versiones de la imagen, sólo se bajan las capas que difieren de la primera imagen descargada.**