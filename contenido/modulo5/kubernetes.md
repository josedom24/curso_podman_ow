# Generación de un archivo YAML de Kubernetes con Podman

Kubernetes es un orquestador de contenedores muy usado en nuestro días. Básicamente, Kubernetes controla la ejecución de Pods en distintos nodos de un clúster de servidores.

Una de las características más importantes de Podman es la capacidad de generar recursos Kubernetes en formato YAML. Además Podman puede crear contenedores y Pods a partir de ficheros YAML de Kubernetes. De está forma con los subcomandos de la instrucción `podman kube` podremos gestionar manifiestos YAMLd e Kubernetes desde Podman.

## Generación de recursos YAML de Kubernetes a partir de contenedores

Podman puede generar la definición en formato de varios recursos de Kubernetes: Pod, Service y PersistentVolumeClaim:

* **Pod**: es un recurso de Kubernetes que nos permite la definición de un Pod.
* **Service**: es un recurso de Kubernetes que nos permite el acceso al Pod.
* **PersistentVolumeClaim**: Es un recurso de Kubernetes que nos permite la creación de un volumen en Kubernetes. Este recurso se podrá crear si usamos volúmenes en Podman. si utilizamos bind mount para configurar el almacenamiento, en la definición del Pod se añadirá un volumen de tipo *hostPath* (es decir, un directorio en el nodo del clúster de Kubernetes).

Veamos algunos ejemplos. En primer lugar vamos a crear un contenedor con un servidor web apache2:

```
$ podman run -d --name webserver -p 8080:8080 quay.io/fedora/httpd-24:2.4
```

A partir de este contenedor vamos a crear los ficheros YAML de definición de recursos de Kubernetes, vamos a crear la definición de un Pod y la definición de un Service (para llo ponemos la opción `-s`) y vamos a guardarlo en un fichero (opción `-f`):

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

Este fichero, donde se definen los dos recursos que hemos mencionado, lo podría ejecutar en un clúster de kubernetes con la instrucción:

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

Si tenemos un contenedor donde hemos montado un volumen:

```
$ podman run -d --name webserver2 -p 8081:8080 -v volweb:/var/www/html quay.io/fedora/httpd-24:2.4
```

Podemos generar, además el recurso PersistantVolumeClaim, para solicitar almacenamiento en el clúster de Kubernetes, para ello indicamos también el nombre del volumen:

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

Para el primer ejemplo partimos del ejemplo anterior donde hemos **Deplegado WordPress + MariaDB en un Pod** utilizando bind mount para conseguir la persistencia.

```
$ sudo podman pod ps --ctr-names
POD ID        NAME        STATUS      CREATED        INFRA ID      NAMES
49cdd1ad01b6  pod-wp-bd   Running     5 minutes ago  e78a5bb025f7  49cdd1ad01b6-infra,servidor_mariadb,servidor_wp
```

En este caso se va a generar un Pod con dos contenedores y con volúmenes del tipo hostPath:

```
$ sudo podman kube generate -s -f wp-mariadb-pod.yaml pod-wp-bd 
```

Y el fichero `wp-mariadb-pod.yaml` quedaría:

```yaml
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-4.9.4
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2024-04-05T15:44:39Z"
  labels:
    app: pod-wp-bd
  name: pod-wp-bd
spec:
  ports:
  - name: "80"
    nodePort: 31032
    port: 80
    targetPort: 80
  selector:
    app: pod-wp-bd
  type: NodePort
---
apiVersion: v1
kind: Pod
metadata:
  annotations:
    bind-mount-options: /home/fedora/wp/cms:Z
  creationTimestamp: "2024-04-05T15:44:39Z"
  labels:
    app: pod-wp-bd
  name: pod-wp-bd
spec:
  containers:
  - args:
    - mariadbd
    env:
    - name: MARIADB_USER
      value: user_wp
    - name: MARIADB_PASSWORD
      value: asdasd
    - name: MARIADB_ROOT_PASSWORD
      value: asdasd
    - name: MARIADB_DATABASE
      value: bd_wp
    image: docker.io/library/mariadb:latest
    name: servidormariadb
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /var/lib/mysql
      name: home-fedora-wp-data-host-0
  - args:
    - apache2-foreground
    env:
    - name: WORDPRESS_DB_HOST
      value: 127.0.0.1
    - name: WORDPRESS_DB_USER
      value: user_wp
    - name: WORDPRESS_DB_NAME
      value: bd_wp
    - name: WORDPRESS_DB_PASSWORD
      value: asdasd
    image: docker.io/library/wordpress:latest
    name: servidorwp
    volumeMounts:
    - mountPath: /var/www/html
      name: home-fedora-wp-cms-host-0
  volumes:
  - hostPath:
      path: /home/fedora/wp/data
      type: Directory
    name: home-fedora-wp-data-host-0
  - hostPath:
      path: /home/fedora/wp/cms
      type: Directory
    name: home-fedora-wp-cms-host-0
```

En 

```
$ sudo podman pod ps --ctr-names
POD ID        NAME        STATUS      CREATED             INFRA ID      NAMES
5bd8b8618742  pod-wp      Running     About a minute ago  060087c84d1b  5bd8b8618742-infra,servidor_wp
5725c5fbc7a9  pod-bd      Running     About a minute ago  f84f780b98b0  5725c5fbc7a9-infra,servidor_mariadb
```


```
$ sudo podman kube generate -s -f wp-mariadb-multipod.yaml pod-wp pod-bd vol-wp vol-data
```

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
  creationTimestamp: "2024-04-05T15:56:19Z"
  name: vol-wp
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
status: {}
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    volume.podman.io/driver: local
  creationTimestamp: "2024-04-05T15:56:19Z"
  name: vol-data
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
status: {}
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2024-04-05T15:56:19Z"
  labels:
    app: pod-wp
  name: pod-wp
spec:
  ports:
  - name: "80"
    nodePort: 30039
    port: 80
    targetPort: 80
  selector:
    app: pod-wp
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2024-04-05T15:56:19Z"
  labels:
    app: pod-bd
  name: pod-bd
spec:
  ports:
  - name: "3306"
    nodePort: 31693
    port: 3306
    targetPort: 3306
  selector:
    app: pod-bd
  type: NodePort
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-04-05T15:56:19Z"
  labels:
    app: pod-wp
  name: pod-wp
spec:
  containers:
  - args:
    - apache2-foreground
    env:
    - name: WORDPRESS_DB_PASSWORD
      value: asdasd
    - name: WORDPRESS_DB_USER
      value: user_wp
    - name: WORDPRESS_DB_NAME
      value: bd_wp
    - name: WORDPRESS_DB_HOST
      value: pod-bd
    image: docker.io/library/wordpress:latest
    name: servidorwp
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /var/www/html
      name: vol-wp-pvc
  volumes:
  - name: vol-wp-pvc
    persistentVolumeClaim:
      claimName: vol-wp
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-04-05T15:56:19Z"
  labels:
    app: pod-bd
  name: pod-bd
spec:
  containers:
  - args:
    - mariadbd
    env:
    - name: MARIADB_ROOT_PASSWORD
      value: asdasd
    - name: MARIADB_DATABASE
      value: bd_wp
    - name: MARIADB_USER
      value: user_wp
    - name: MARIADB_PASSWORD
      value: asdasd
    image: docker.io/library/mariadb:latest
    name: servidormariadb
    ports:
    - containerPort: 3306
    volumeMounts:
    - mountPath: /var/lib/mysql
      name: vol-data-pvc
  volumes:
  - name: vol-data-pvc
    persistentVolumeClaim:
      claimName: vol-data
```