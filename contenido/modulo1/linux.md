# Instalación de Podman en Linux

En la [página oficial sobre instalación de Podman](https://podman.io/docs/installation) en Linux puedes encontrar las instrucciones para la instalación de Podman en las distintas distribuciones Linux. Pongamos algunos ejemplos:

En distribuciones Debian/Linux:

```
$ sudo apt install podman
```

En la distribución Fedora:

```
$ sudo dnf install podman
```

Hay que tener en cuenta que en este curso se va trabajar con la versión 4.X de Podman, por lo que se sugieren los siguientes sistemas operativos:

* Fedora 39, que nos ofrece Podman 4.9.4 (esta distribución es la que voy a usar durante el curso).
* Debian 12 Bookworm, que nos ofrece Podman 4.3.1.
* Ubuntu 23.04 y 23.10, que nos ofrece Podman 4.3.1.
* Ubuntu 24.04, que nos ofrece Podman 4.9.3.
* Fedora 40, nos ofrece Podman 5.0.1.

Una vez terminada la instalación, podemos comprobar la información de la versión instalada, ejecutando:

```
$ podman version
$ podman info
```
 