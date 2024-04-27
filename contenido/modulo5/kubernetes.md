# Generación de un archivo YAML de Kubernetes con Podman

Kubernetes es un orquestador de contenedores muy usado en nuestros días. Básicamente, Kubernetes controla la ejecución de Pods en distintos nodos de un clúster de servidores.

Una de las características más importantes de Podman es la capacidad de generar recursos Kubernetes en formato YAML. Además Podman puede crear contenedores y Pods a partir de ficheros YAML de Kubernetes. De está forma con los subcomandos de la instrucción `podman kube` podremos gestionar manifiestos YAML de Kubernetes desde Podman.

## Generación de recursos YAML de Kubernetes a partir de contenedores

Puedes encontrar los ficheros que vamos a generar en este apartado en el directorio `modulo5/kube` del [Repositorio con el código de los ejemplos](https://github.com/josedom24/ejemplos_curso_podman_ow).

Podman puede generar la definición en formato de varios recursos de Kubernetes: Pod, Service y PersistentVolumeClaim:

* **Pod**: es un recurso de Kubernetes que nos permite la definición de un Pod.
* **Service**: es un recurso de Kubernetes que nos permite el acceso al Pod.
* **PersistentVolumeClaim**: Es un recurso de Kubernetes que nos permite la creación de un volumen en Kubernetes. Este recurso se creará a pertir de los volúmenes de Podman. Si utilizamos bind mount para configurar el almacenamiento, en la definición del Pod se añadirá un volumen de tipo *hostPath* (es decir, un directorio en el nodo del clúster de Kubernetes).

Veamos algunos ejemplos. En primer lugar vamos a crear un contenedor con un servidor web apache2:

```
$ podman run -d --name webserver -p 8080:8080 quay.io/fedora/httpd-24:2.4
```

A partir de este contenedor vamos a crear los ficheros YAML de definición de recursos de Kubernetes, vamos a crear la definición de un Pod y la definición de un Service (para ello ponemos la opción `-s`) y vamos a guardarlo en un fichero (opción `-f`):

```
$ podman kube generate -s -f web-pod.yaml webserver
```

El contenido del fichero `web-pod.yaml` es el siguiente:

```yaml
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-4.9.4
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2024-04-05T06:53:21Z"
  labels:
    app: webserver-pod
  name: webserver-pod
spec:
  ports:
  - name: "8080"
    nodePort: 30208
    port: 8080
    targetPort: 8080
  selector:
    app: webserver-pod
  type: NodePort
---
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
    securityContext:
      runAsNonRoot: true
```

Este fichero, donde se definen los dos recursos que hemos mencionado, lo podría ejecutar en un clúster de Kubernetes con la instrucción:

```
$ kubectl apply -f web-pod.yaml 
```

Y obtener los recursos que se han creado:

```
$ kubectl get all
NAME                READY   STATUS    RESTARTS   AGE
pod/webserver-pod   1/1     Running   0          51s

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP          57d
service/webserver-pod   NodePort    10.108.230.189   <none>        8080:30208/TCP   52s
```

## Generación de recursos YAML de Kubernetes a partir de contenedores con volúmenes

Si tenemos un contenedor donde hemos montado un volumen:

```
$ podman run -d --name webserver2 -p 8081:8080 -v volweb:/var/www/html quay.io/fedora/httpd-24:2.4
```

Podemos generar, además el recurso `PersistantVolumeClaim`, para solicitar almacenamiento en el clúster de Kubernetes, para ello indicamos también el nombre del volumen:

```
$ podman kube generate -s -f web-pod2.yaml webserver2 volweb
```

El fichero `web-pod2.yaml` contendría, además de la definición de los recursos anteriores, la definición del PersistantVolumeClaim:

```yaml
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-4.9.4
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    volume.podman.io/driver: local
  creationTimestamp: "2024-04-05T07:06:45Z"
  name: volweb
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
status: {}
---
...
```

## Generación de recursos YAML de Kubernetes a partir de Pods

### Ejemplo: Desplegado WordPress + MariaDB en un Pod

Para el primer ejemplo partimos del escenario que creamos en el apartado **Ejemplo: Desplegado WordPress + MariaDB en un Pod**:

```
$ podman pod ps --ctr-names
POD ID        NAME               STATUS      CREATED         INFRA ID      NAMES
d1d937d15358  wordpress-pod      Running     3 minutes ago   27326c5e4a67  d1d937d15358-infra,db,wordpress
```

En este caso se va a generar un Pod con dos contenedores y dos PersistentVolumeClaim correspondientes a los volúmenes:

```
$ podman kube generate wordpress-pod wpvol dbvol -f wp-mariadb-pod.yaml
```

El fichero `wp-mariadb-pod.yaml` que hemos generado lo puedes visualizar en este [enlace](https://raw.githubusercontent.com/josedom24/ejemplos_curso_podman_ow/main/modulo5/kube/wp-mariadb-pod.yaml).

Podemos desplegar el fichero en nuestro clúster de Kubernetes:

```
$ kubectl apply -f wp-mariadb-pod.yaml 
$ kubectl expose --type=NodePort pod/wordpress-pod

$ kubectl get all,pvc
NAME                READY   STATUS    RESTARTS   AGE
pod/wordpress-pod   2/2     Running   0          10s

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP        80d
service/wordpress-pod   NodePort    10.103.123.157   <none>        80:32747/TCP   3s

NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/dbvol   Bound    pvc-8de76765-48f2-46ab-ba91-f0b9e6b1a3a5   1Gi        RWO            standard       10s
persistentvolumeclaim/wpvol   Bound    pvc-c15a471d-3029-467e-88fe-6ed987e65ea7   1Gi        RWO            standard       10s
```

### Ejemplo: Despliegue de WordPress + MariaDB en un escenario multipod


Para el segundo ejemplo partimos del escenario que creamos en el apartado **Ejemplo: Despliegue de WordPress + MariaDB en un escenario multipod**: 

```
$ podman pod ps --ctr-names
POD ID        NAME               STATUS      CREATED        INFRA ID      NAMES
4539e047ef62  mariadb-pod        Running     2 minutes ago  7ee82573c019  4539e047ef62-infra,db
9646085b179e  wordpress-pod      Running     8 minutes ago  4274fc776781  9646085b179e-infra,wordpress
```


```
$ podman kube generate wordpress-pod mariadb-pod wpvol dbvol -f wp-mariadb-multipod.yaml 
```

El fichero `wp-mariadb-multipod.yaml` que hemos generado lo puedes visualizar en este [enlace](https://raw.githubusercontent.com/josedom24/ejemplos_curso_podman_ow/main/modulo5/kube/wp-mariadb-multipod.yaml).

Podemos desplegar el fichero en nuestro clúster de Kubernetes:

```
$ kubectl apply -f wp-mariadb-multipod.yaml 
$ kubectl expose --type=NodePort pod/wordpress-pod
$ kubectl expose --type=ClusterIP pod/mariadb-pod

$ kubectl get all,pvc
NAME                READY   STATUS    RESTARTS   AGE
pod/mariadb-pod     1/1     Running   0          32s
pod/wordpress-pod   1/1     Running   0          32s

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP        80d
service/mariadb-pod     ClusterIP   10.101.138.22    <none>        3306/TCP       1s
service/wordpress-pod   NodePort    10.103.123.157   <none>        80:32747/TCP   6s

NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/dbvol   Bound    pvc-c0d55626-abe2-4dbc-9cb2-f313ae58d8de   1Gi        RWO            standard       32s
persistentvolumeclaim/wpvol   Bound    pvc-c12fe215-546c-4c39-a7d9-fe228d9cebb1   1Gi        RWO            standard       32s
```
