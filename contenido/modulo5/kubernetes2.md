# Ejecutando recursos de Kubernetes en Podman

Además de poder generar definiciones de recursos de Kubernetes en fichero YAML. En Podman también podemos ejecutar Pods y contenedores a partir de manifiestos YAML de Kubernetes.


```
$ sudo podman kube play wp-mariadb-pod.yaml
...
Volumes:
dbvol
wpvol
Pod:
cd2d6fc0916ba73e769562382b80464108b1305e28db9847f963212c608a43e8
Containers:
93360a599956da91011c1b720e2fe599e49afd898810c19bba079a4ddf828b1c
42621e47f8155ae40c1d3a7139279d8f23c1c40be21bbc65b742b511cced34c9

$ sudo podman pod ps --ctr-names
POD ID        NAME           STATUS      CREATED        INFRA ID      NAMES
cd2d6fc0916b  wordpress-pod  Running     2 minutes ago  bee585832127  cd2d6fc0916b-infra,wordpress-pod-db,wordpress-pod-wordpress

$ sudo podman volume ls
DRIVER      VOLUME NAME
local       dbvol
local       wpvol
```

```
$ sudo podman kube play wp-mariadb-multipod.yaml
Volumes:
dbvol
wpvol
Pod:
0e07102382de6961e899eba77b60614dd449fbc72b6ff64a38a918bf565492d2
Container:
13d2ab8682dc6fcb9fb47db6c8b9eb12a36e57a8ccf62d1c32c79ff62e8d2beb

Pod:
b1dd4010698aeda650b9ba3c808de81ad16b1d4845a68be0a3d6013e18c217b1
Container:
7bc05f2f048843b03bba694fc4802d299c7c5454721123029f31f9b167c18379

$ sudo podman pod ps --ctr-names
POD ID        NAME           STATUS      CREATED         INFRA ID      NAMES
b1dd4010698a  mariadb-pod    Running     34 seconds ago  5cf7cb140f7a  b1dd4010698a-infra,mariadb-pod-db
0e07102382de  wordpress-pod  Running     40 seconds ago  9ce1886ab6cc  0e07102382de-infra,wordpress-pod-wordpress

$ sudo podman volume ls
DRIVER      VOLUME NAME
local       dbvol
local       wpvol
```
