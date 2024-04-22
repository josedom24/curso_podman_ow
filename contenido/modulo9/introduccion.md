# Introducción a Buildah y Skopeo

## Buildah

[Buildah](https://buildah.io/) facilita la construcción de imágenes de contenedores OCI. Nos permite crear imágenes OCI de varias formas:

* Puede crear un **contenedor de trabajo** utilizando una **imagen como punto de partida**. Puede modificar dicho contenedor y a partir de él crear una nueva imagen.
* Puede crear un **contenedor de trabajo** vacío (*from scratch*), montar el sistema de archivo e inicializarlo con un sistema base.
* Puede crear imágenes OCI a partir de fichero `Containerfile` o `Dockerfile`.

Las características fundamentales de Buildah son:

* **Construcción de imágenes sin demonio**: Buildah permite construir imágenes de contenedores sin necesidad de un demonio. 
* **Construcción de imágenes sin usuario root**: Cualquier usuario sin privilegios puede construir imágenes con Buildah.
* **Compatibilidad con el estándar OCI**: Buildah cumple con el estándar Open Container Initiative (OCI), lo que garantiza que las imágenes y contenedores creados sean interoperables con otras herramientas y plataformas que siguen este estándar.
* **Manipulación de imágenes y contenedores**: Además de la construcción de imágenes, Buildah facilita la manipulación de imágenes y contenedores. Puedes modificar imágenes existentes, añadir o eliminar capas, y realizar otras operaciones para personalizar y mantener tus imágenes.
* **Integración con herramientas de automatización**: Buildah se integra fácilmente en flujos de trabajo de automatización y sistemas de construcción. 
* **Soporte para múltiples formatos de salida**: Buildah puede generar imágenes en varios formatos de salida, incluyendo Docker, OCI, y otros formatos compatibles.

## Skopeo

[Skopeo](https://github.com/containers/skopeo) es una herramienta que se utiliza para inspeccionar, copiar y transportar imágenes de contenedor entre diferentes registros. Está diseñada para trabajar con imágenes en formato OCI (Open Container Initiative) y Docker, permitiendo a los usuarios realizar varias operaciones relacionadas con imágenes de contenedores de manera eficiente y sin necesidad de un demonio.

Las características principales son:

* **Inspección de imágenes**: Skopeo permite a los usuarios inspeccionar imágenes de contenedor almacenadas en registros remotos sin necesidad de bajarlas a un registro local.
* **Copia de imágenes**: Con Skopeo, los usuarios pueden copiar imágenes de un registro a otro, lo que facilita la migración de imágenes entre diferentes entornos.
* **Transporte de imágenes**: Skopeo utiliza los distintos transportes de imágenes para distribuirlas. 
* **Compatibilidad con múltiples formatos**: Skopeo es compatible con varios formatos de imágenes de contenedores, incluyendo Docker y OCI.
* **Integración con sistemas de automatización**: Skopeo se puede integrar fácilmente en flujos de trabajo de sistema de integración y despliegue continúo.

## Instalación de Buildah y Skopeo

Puedes encontrar el método de instalación de Buildah en su [página oficial](https://github.com/containers/buildah/blob/main/install.md). Por ejemplo:

Para sistemas Debian/Ubuntu:

```
$ sudo apt install buildah
```

Para sistemas Fedora:

```
$ sudo dnf install buildah
```

De manera similar, pueder consultar los métodos de instalación de Skopeo en su [página oficial](https://github.com/containers/skopeo/blob/main/install.md). Por ejemplo:

Para sistemas Debian/Ubuntu:

```
$ sudo apt install skopeo
```

Para sistemas Fedora:

```
$ sudo dnf install skopeo
```
