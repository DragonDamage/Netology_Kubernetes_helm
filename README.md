# Задание 1. Подготовить Helm-чарт для приложения

1. Необходимо упаковать приложение в чарт для деплоя в разные окружения. 
2. Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.
3. В переменных чарта измените образ приложения для изменения версии.

---

# Выполнение задания 1

```bash
bogatyrevam@vm:~/Desktop/7-helm$ nano chart.yaml
bogatyrevam@vm:~/Desktop/7-helm$ nano values.yaml
bogatyrevam@vm:~/Desktop/7-helm$ nano values-v2.yaml
bogatyrevam@vm:~/Desktop/7-helm$ mkdir templates
bogatyrevam@vm:~/Desktop/7-helm$ nano templates/values-v3.yaml
bogatyrevam@vm:~/Desktop/7-helm$ nano templates/nginx-deployment.yaml
bogatyrevam@vm:~/Desktop/7-helm$ nano templates/network-multitool-deployment.yaml
```

### Chart.yaml
```yaml
apiVersion: v2
name: my-app
description: Helm chart for deploying nginx and wbitt/network-multitool
version: 0.1.0
appVersion: "1.0.0"
```

### values.yaml
```yaml
nginx:
  image: nginx:1.25.3
  replicas: 2
  service:
    type: ClusterIP
    port: 80

networkMultitool:
  image: wbitt/network-multitool:latest
  replicas: 1
  service:
    type: ClusterIP
    port: 8080
```

### values-v2.yaml
```yaml
nginx:
  image: nginx:1.24.0
  replicas: 2
  service:
    type: ClusterIP
    port: 80

networkMultitool:
  image: wbitt/network-multitool:latest
  replicas: 1
  service:
    type: ClusterIP
    port: 8080
```

### values-v3.yaml
```yaml
nginx:
  image: nginx:latest
  replicas: 2
  service:
    type: ClusterIP
    port: 80

networkMultitool:
  image: wbitt/network-multitool
  replicas: 1
  service:
    type: ClusterIP
    port: 8080
```

### templates/nginx-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-nginx
  labels:
    app: {{ .Chart.Name }}
    component: nginx
spec:
  replicas: {{ .Values.nginx.replicas }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
      component: nginx
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
        component: nginx
    spec:
      containers:
      - name: nginx
        image: {{ .Values.nginx.image }}
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-nginx
spec:
  type: {{ .Values.nginx.service.type }}
  ports:
  - port: {{ .Values.nginx.service.port }}
    targetPort: 80
  selector:
    app: {{ .Chart.Name }}
    component: nginx
```

### templates/network-multitool-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-network-multitool
  labels:
    app: {{ .Chart.Name }}
    component: network-multitool
spec:
  replicas: {{ .Values.networkMultitool.replicas }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
      component: network-multitool
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
        component: network-multitool
    spec:
      containers:
      - name: network-multitool
        image: {{ .Values.networkMultitool.image }}
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-network-multitool
spec:
  type: {{ .Values.networkMultitool.service.type }}
  ports:
  - port: {{ .Values.networkMultitool.service.port }}
    targetPort: 8080
  selector:
    app: {{ .Chart.Name }}
    component: network-multitool
```

------
### Задание 2. Запустить две версии в разных неймспейсах

1. Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
2. Одну версию в namespace=app1, вторую версию в том же неймспейсе, третью версию в namespace=app2.
3. Продемонстрируйте результат.

---

# Выполнение задания 2
```bash
root@vm:/home/bogatyrevam/Desktop/7-helm# microk8s helm3 lint .
==> Linting .
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

```bash
root@vm:/home/bogatyrevam/Desktop/7-helm# microk8s kubectl create namespace app1
namespace/app1 created
```

```bash
root@vm:/home/bogatyrevam/Desktop/7-helm# microk8s helm3 install my-app-v1 . -n app1
NAME: my-app-v1
LAST DEPLOYED: Mon Jul  6 18:02:56 2026
NAMESPACE: app1
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

```bash
root@vm:/home/bogatyrevam/Desktop/7-helm# microk8s kubectl get all -n app1
NAME                                               READY   STATUS    RESTARTS   AGE
pod/my-app-v1-network-multitool-74f6fb8fd4-s6jtg   1/1     Running   0          61s
pod/my-app-v1-nginx-66b9599867-4nlgq               1/1     Running   0          61s
pod/my-app-v1-nginx-66b9599867-lcgfr               1/1     Running   0          61s

NAME                                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/my-app-v1-network-multitool   ClusterIP   10.152.183.247   <none>        8080/TCP   63s
service/my-app-v1-nginx               ClusterIP   10.152.183.100   <none>        80/TCP     63s

NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-app-v1-network-multitool   1/1     1            1           61s
deployment.apps/my-app-v1-nginx               2/2     2            2           61s

NAME                                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/my-app-v1-network-multitool-74f6fb8fd4   1         1         1       61s
replicaset.apps/my-app-v1-nginx-66b9599867               2         2         2       61s
```

```bash
root@vm:/home/bogatyrevam/Desktop/7-helm# microk8s helm3 install my-app-v2 . -n app1 -f values-v2.yaml
NAME: my-app-v2
LAST DEPLOYED: Mon Jul  6 18:05:11 2026
NAMESPACE: app1
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

```bash
root@vm:/home/bogatyrevam/Desktop/7-helm# microk8s kubectl get all -n app1
NAME                                               READY   STATUS              RESTARTS   AGE
pod/my-app-v1-network-multitool-74f6fb8fd4-s6jtg   1/1     Running             0          2m45s
pod/my-app-v1-nginx-66b9599867-4nlgq               1/1     Running             0          2m45s
pod/my-app-v1-nginx-66b9599867-lcgfr               1/1     Running             0          2m45s
pod/my-app-v2-network-multitool-74f6fb8fd4-fls9p   1/1     Running             0          30s
pod/my-app-v2-nginx-6ddc9c78bf-bsc7r               0/1     ContainerCreating   0          30s
pod/my-app-v2-nginx-6ddc9c78bf-h8jsq               0/1     ContainerCreating   0          30s

NAME                                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/my-app-v1-network-multitool   ClusterIP   10.152.183.247   <none>        8080/TCP   2m47s
service/my-app-v1-nginx               ClusterIP   10.152.183.100   <none>        80/TCP     2m47s
service/my-app-v2-network-multitool   ClusterIP   10.152.183.35    <none>        8080/TCP   32s
service/my-app-v2-nginx               ClusterIP   10.152.183.206   <none>        80/TCP     32s

NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-app-v1-network-multitool   1/1     1            1           2m45s
deployment.apps/my-app-v1-nginx               2/2     2            2           2m45s
deployment.apps/my-app-v2-network-multitool   1/1     1            1           31s
deployment.apps/my-app-v2-nginx               0/2     2            0           31s

NAME                                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/my-app-v1-network-multitool-74f6fb8fd4   1         1         1       2m45s
replicaset.apps/my-app-v1-nginx-66b9599867               2         2         2       2m45s
replicaset.apps/my-app-v2-network-multitool-74f6fb8fd4   1         1         1       30s
replicaset.apps/my-app-v2-nginx-6ddc9c78bf               2         2         0       30s
```

```bash
root@vm:/home/bogatyrevam/Desktop/7-helm# microk8s kubectl create namespace app2
namespace/app2 created
```

```bash
root@vm:/home/bogatyrevam/Desktop/7-helm# microk8s helm3 install my-app-v3 . -n app2 -f values-v3.yaml
NAME: my-app-v3
LAST DEPLOYED: Mon Jul  6 18:08:01 2026
NAMESPACE: app2
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

```bash
root@vm:/home/bogatyrevam/Desktop/7-helm# microk8s kubectl get all -n app2
NAME                                               READY   STATUS    RESTARTS   AGE
pod/my-app-v3-network-multitool-669fd8fbc9-nn6bn   1/1     Running   0          31s
pod/my-app-v3-nginx-dd75567dc-mh6qf                1/1     Running   0          31s
pod/my-app-v3-nginx-dd75567dc-rqvzx                1/1     Running   0          31s

NAME                                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/my-app-v3-network-multitool   ClusterIP   10.152.183.195   <none>        8080/TCP   34s
service/my-app-v3-nginx               ClusterIP   10.152.183.29    <none>        80/TCP     34s

NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-app-v3-network-multitool   1/1     1            1           32s
deployment.apps/my-app-v3-nginx               2/2     2            2           32s

NAME                                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/my-app-v3-network-multitool-669fd8fbc9   1         1         1       31s
replicaset.apps/my-app-v3-nginx-dd75567dc                2         2         2       31s
```

```bash
root@vm:/home/bogatyrevam/Desktop/7-helm# microk8s kubectl get all -A
NAMESPACE              NAME                                                        READY   STATUS    RESTARTS      AGE
app1                   pod/my-app-v1-network-multitool-74f6fb8fd4-s6jtg            1/1     Running   0             6m27s
app1                   pod/my-app-v1-nginx-66b9599867-4nlgq                        1/1     Running   0             6m27s
app1                   pod/my-app-v1-nginx-66b9599867-lcgfr                        1/1     Running   0             6m27s
app1                   pod/my-app-v2-network-multitool-74f6fb8fd4-fls9p            1/1     Running   0             4m12s
app1                   pod/my-app-v2-nginx-6ddc9c78bf-bsc7r                        1/1     Running   0             4m12s
app1                   pod/my-app-v2-nginx-6ddc9c78bf-h8jsq                        1/1     Running   0             4m12s
app2                   pod/my-app-v3-network-multitool-669fd8fbc9-nn6bn            1/1     Running   0             81s
app2                   pod/my-app-v3-nginx-dd75567dc-mh6qf                         1/1     Running   0             81s
app2                   pod/my-app-v3-nginx-dd75567dc-rqvzx                         1/1     Running   0             81s

NAMESPACE              NAME                                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
app1                   service/my-app-v1-network-multitool            ClusterIP   10.152.183.247   <none>        8080/TCP                        6m30s
app1                   service/my-app-v1-nginx                        ClusterIP   10.152.183.100   <none>        80/TCP                          6m30s
app1                   service/my-app-v2-network-multitool            ClusterIP   10.152.183.35    <none>        8080/TCP                        4m15s
app1                   service/my-app-v2-nginx                        ClusterIP   10.152.183.206   <none>        80/TCP                          4m15s
app2                   service/my-app-v3-network-multitool            ClusterIP   10.152.183.195   <none>        8080/TCP                        85s
app2                   service/my-app-v3-nginx                        ClusterIP   10.152.183.29    <none>        80/TCP                          85s

NAMESPACE              NAME                                                   READY   UP-TO-DATE   AVAILABLE   AGE
app1                   deployment.apps/my-app-v1-network-multitool            1/1     1            1           6m29s
app1                   deployment.apps/my-app-v1-nginx                        2/2     2            2           6m29s
app1                   deployment.apps/my-app-v2-network-multitool            1/1     1            1           4m15s
app1                   deployment.apps/my-app-v2-nginx                        2/2     2            2           4m15s
app2                   deployment.apps/my-app-v3-network-multitool            1/1     1            1           84s
app2                   deployment.apps/my-app-v3-nginx                        2/2     2            2           84s

NAMESPACE              NAME                                                              DESIRED   CURRENT   READY   AGE
app1                   replicaset.apps/my-app-v1-network-multitool-74f6fb8fd4            1         1         1       6m29s
app1                   replicaset.apps/my-app-v1-nginx-66b9599867                        2         2         2       6m29s
app1                   replicaset.apps/my-app-v2-network-multitool-74f6fb8fd4            1         1         1       4m14s
app1                   replicaset.apps/my-app-v2-nginx-6ddc9c78bf                        2         2         2       4m14s
app2                   replicaset.apps/my-app-v3-network-multitool-669fd8fbc9            1         1         1       83s
app2                   replicaset.apps/my-app-v3-nginx-dd75567dc                         2         2         2       83s
```







