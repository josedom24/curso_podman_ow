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

------------------
De la página:https://ravichaganti.com/blog/2022-10-28-understanding-container-images-oci-image-specification/


Alrededor de 2015, cuando la contenedorización empezó a convertirse en la corriente dominante, algunas empresas empezaron a crear herramientas competidoras. Rkt fue una de ellas. La gente de CoreOS creó el formato Application Container Image (ACI) y las especificaciones Application Container Execution (ACE) y comenzó a implementar herramientas. Al mismo tiempo, Docker ya había conseguido llevar los contenedores a las masas. Así, la industria vio la oportunidad de eliminar la fragmentación en el ecosistema de contenedores y trabajó con Docker, CoreOS y otras empresas para crear Open Container Initiative (OCI) con el fin de desarrollar estándares en torno a formatos, tiempos de ejecución y registros de contenedores. La mayoría o todos los motores y herramientas de contenedores aplican las especificaciones de la OCI.


La especificación de imagen OCI contiene un índice de imagen, un manifiesto de imagen, un conjunto de capas del sistema de archivos y una configuración. Aunque parezca mucho, en realidad es bastante sencillo.

La figura anterior representa la asociación entre las distintas partes de la especificación de imagen OCI. Comienza con un índice de imagen opcional.

* Índice de imagen: un índice de manifiestos que apuntan a imágenes para diferentes plataformas / arquitecturas.
* Manifiesto de imagen: contiene referencias al JSON de configuración de la imagen y un conjunto de capas del sistema de archivos utilizadas para crear la imagen. El contenido de un manifiesto de imagen es específico de una plataforma y versión de SO.
  * El elemento config hace referencia a la representación JSON de los metadatos de la imagen, los parámetros de ejecución para crear un contenedor y las capas del sistema de archivos que se utilizarán al crear el contenedor.
  * ....


