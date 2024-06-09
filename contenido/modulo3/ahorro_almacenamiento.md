# Ahorro de espacio de almacenamiento

La estructura de almacenamiento que hemos explicado favorece el ahorro de espacio en disco ocupado por las imágenes. Si al descargar una imagen a nuestro registro local, esta formada por alguna capa que ya tenemos almacenada de otra imagen, esta capa no se descargará. Veamos un ejemplo:

Si a continuación bajamos otra versión de la misma imagen:

```
# podman pull quay.io/centos7/httpd-24-centos7:20230712
Trying to pull quay.io/centos7/httpd-24-centos7:20230712...
Getting image source signatures
Copying blob c61d16cfe03e skipped: already exists  
Copying blob 06c7e4737942 skipped: already exists  
Copying blob cad88c5d6507 done   | 
Copying config 6211883c1e done   | 
Writing manifest to image destination
6211883c1ed7ec96a12bbc9b214e70e5af406361db435905e85ba88b1483645b
```

Vemos cómo dos de las tres capas no se han descargados, porque son las mismas que teníamos ya descargadas. Si vemos el fichero de manifiesto donde se indican las capas de esta nueva imagen:

```
# cat overlay-images/6211883c1ed7ec96a12bbc9b214e70e5af406361db435905e85ba88b1483645b/manifest | jq
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
  "config": {
    "mediaType": "application/vnd.docker.container.image.v1+json",
    "size": 15279,
    "digest": "sha256:6211883c1ed7ec96a12bbc9b214e70e5af406361db435905e85ba88b1483645b"
  },
  "layers": [
    {
      "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
      "size": 78951426,
      "digest": "sha256:c61d16cfe03e7bfb4e7e312f09fb17a815be72096544133320058ee6ce55d0b2"
    },
    {
      "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
      "size": 10435798,
      "digest": "sha256:06c7e47379429b2a921140524d1596e2c2bf8bc7b29fa9df0ee73c91f5b4c24f"
    },
    {
      "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
      "size": 50603263,
      "digest": "sha256:cad88c5d6507a80282a7942c790b0290aef5d302b790d988714de3535c7f8eb9"
    }
  ]
}
```
Vemos que las dos primeras capas coinciden con las de la imagen anterior y por tanto no se han descargado. De forma gráfica lo podríamos ver de la siguiente manera:

![images4](img/images4.png)

## Calcular el espacio que ocupan las imágenes

Veamos con más detalle las capas que forman cada una de las imágenes, cuáles se comparten entre las dos imágenes y qué espacio ocupan en el disco duro:

```
# podman image tree quay.io/centos7/httpd-24-centos7:centos7
Image ID: d7af31210b28
Tags:     [quay.io/centos7/httpd-24-centos7:centos7]
Size:     356.5MB
Image Layers
├── ID: c61d16cfe03e Size: 211.8MB
├── ID: 06c7e4737942 Size:  34.3MB
└── ID: 8f001c8d7e00 Size: 110.3MB Top Layer of: [quay.io/centos7/httpd-24-centos7:centos7]

# podman image tree quay.io/centos7/httpd-24-centos7:20230712 
Image ID: 6211883c1ed7
Tags:     [quay.io/centos7/httpd-24-centos7:20230712]
Size:     356.5MB
Image Layers
├── ID: c61d16cfe03e Size: 211.8MB
├── ID: 06c7e4737942 Size:  34.3MB
└── ID: cad88c5d6507 Size: 110.3MB Top Layer of: [quay.io/centos7/httpd-24-centos7:20230712]
```

Comprobamos que cada una de las imágenes ocupan 356.5MB, por lo tanto podríamos pensar que el total de espacio ocupado sería 356.5MB + 356.5MB. Sin embargo, vemos que las dos primeras capas están compartidas por ambas imágenes. Por lo tanto, al descargar la segunda imagen esas dos capas no se han descargado en disco.

Por lo tanto, ¿cuánto han ocupado en total estas dos imágenes en el disco duro? Pues sería 356.5MB de la primera imagen + 110.3MB de la tercera capa de la segunda imagen que es única. El mecanismo de compartir capas entre imágenes hace que se ocupe el menor espacio posible en disco duro, el almacenamiento es muy eficiente.


