### creating custom repo

```
❯ helm create webapp1            #Creating webapp1

❯ tree webapp1

webapp1
├── .helmignore
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

4 directories, 11 files

```

- Chart.yaml : other than changing  name and version, we don't need to do other stuffs here
```
apiVersion: v2
name: webapp1
description: A Helm chart for Kubernetes

type: application

version: 0.1.0

appVersion: "1.16.0"
```

- values.yaml : empty this and start from scratch

- charts/ : we put charts dependencies(on other charts) here

- templates/ : we make most of the changes here, also we can remove everthing from this dir and start from scratch

```
# cat configmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmapv1.0
  namespace: default
data:
  BG_COLOR: '#12181b'
  FONT_COLOR: '#FFFFFF'
  CUSTOM_HEADER: 'Customized with a configmap!'

# cat deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydeployment
  namespace: default
  labels:
    app: mywebapp
spec:
  replicas: 5
  selector:
    matchLabels:
      app: mywebapp
      tier: frontend
  template:
    metadata:
      labels:
        app: mywebapp
        tier: frontend
    spec: # Pod spec
      containers:
      - name: mycontainer
        image: devopsjourney1/mywebapp:latest
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: myconfigmapv1.0
        resources:
          requests:
            memory: "16Mi"
            cpu: "50m"    # 50 milli cores (1/20 CPU)
          limits:
            memory: "128Mi" # 128 mebibytes
            cpu: "100m"

# cat service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mywebapp
  namespace: default
  labels:
    app: mywebapp
spec:
  ports:
  - port: 80
    protocol: TCP
    name: flask
  selector:
    app: mywebapp
    tier: frontend
  type: NodePort
```

helm install mywebapp-release webapp1         # run from the dir where it[webapp1] was created

helm list -a

helm uninstall mywebapp-release

```
# cat deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.appName }}
  namespace: default
  labels:
    app: {{ .Values.appName }}
spec:
  replicas: 5
  selector:
    matchLabels:
      app: {{ .Values.appName }}
      tier: frontend
  template:
    metadata:
      labels:
        app: {{ .Values.appName }}
        tier: frontend
    spec: # Pod spec
      containers:
      - name: mycontainer
        image: devopsjourney1/mywebapp:latest
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: myconfigmapv1.0
        resources:
          requests:
            memory: "16Mi"
            cpu: "50m"    # 50 milli cores (1/20 CPU)
          limits:
            memory: "128Mi" # 128 mebibytes
            cpu: "100m"

# cat service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.appName }}
  namespace: default
  labels:
    app: {{ .Values.appName }}
spec:
  ports:
  - port: 80
    protocol: TCP
    name: flask
  selector:
    app: {{ .Values.appName }}
    tier: frontend
  type: NodePort

```

#### cat values.yaml

```
appName: myhelmapp
```


helm upgrade mywebapp-release webapp1/ --values webapp1/values.yaml

helm ls

```
❯ helm history mywebapp-release
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Sun Jul 21 20:34:12 2024        superseded      webapp1-0.1.0   1.16.0          Install complete
2               Sun Jul 21 21:05:00 2024        deployed        webapp1-0.1.0   1.16.0          Upgrade complete
```

helm upgrade myhelloworld helloworld         #upgrade after changes

helm rollback myhelloworld 1

More changes to the manifests file and values.yaml file
```
# cat values.yaml
appName: myhelmapp

namespace: default

configmap:   
  name: helmappconfigv1.1
  data:
    CUSTOM_HEADER:  "This app was deployed with helm"

image:
  name: devopsjourney1/mywebapp
  tag: latest

```

helm upgrade mywebapp-release webapp1/ --values webapp1/values.yaml

helm ls

```
❯ k get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        49d
myhelmapp    NodePort    10.108.205.89   <none>        80:30860/TCP   111m
❯ curl localhost:30860
<body style='background-color:#12181b;'>    <h1 style='color:#FFFFFF'>This app was deployed with helm</h1>     <img src='https://raw.githubusercontent.com/devopsjourney1/assets/main/devops-journey-banner.png' alt='CUSTOMER_PHOTO'>    <h2 style='color:#FFFFFF;'>Hello World! Served from <b>myhelmapp-5877d878b9-4cmjj</b></h2></body>                                                                                                                                ❯ helm ls
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
mywebapp-release        default         3               2024-07-21 22:55:56.774608 +0545 +0545  deployed        webapp1-0.1.0   1.16.0
```

content inside of webapp1/templates/NOTES.txt is used to pass information during deployment/upgrade of helm chart
```
#cat templates/NOTES.txt

map the nodeport port to curl:[nodeport]
displayed on running `k get svc` to check whether the deployment is working on not from command line
```
helm upgrade mywebapp-release webapp1/ --values webapp1/values.yaml

displayed on the last line of the output
```
❯ helm upgrade mywebapp-release webapp1/ --values webapp1/values.yaml

Release "mywebapp-release" has been upgraded. Happy Helming!
NAME: mywebapp-release
LAST DEPLOYED: Sun Jul 21 23:08:45 2024
NAMESPACE: default
STATUS: deployed
REVISION: 4
TEST SUITE: None
NOTES:
map the nodeport port to curl:[nodeport]
displayed on running `k get svc` to check whether the deployment is working on not from command line
```


### Templating for multiple environments
k create ns dev

k create ns prod

Create values-dev and values-prod for dev and prod environments with only required changes since we'll inherit default values from values.yaml
```
#cat webapp1/values-dev.yaml
namespace: dev

configmap:   
  data:
    CUSTOM_HEADER:  "Deploying in dev, welcome"
```

```
#cat webapp1/values-prod.yaml

namespace: prod

configmap:   
  data:
    CUSTOM_HEADER:  "Deploying in production, welcome"
```

helm install mywebapp-release webapp1/ --values webapp1/values.yaml -f webapp1/values-dev.yaml -n dev

helm install mywebapp-release webapp1/ --values webapp1/values.yaml -f webapp1/values-prod.yaml -n prod

```
❯ helm install mywebapp-release webapp1/ --values webapp1/values.yaml -f webapp1/values-dev.yaml -n dev

NAME: mywebapp-release
LAST DEPLOYED: Sun Jul 21 23:23:36 2024
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
dev
map the nodeport port to `curl localhost:[nodeport]`
displayed on running `k get svc` to check whether the deployment is working on not from command line

```

```
❯ k get pods -A

NAMESPACE     NAME                                     READY   STATUS    RESTARTS   AGE
default       myhelmapp-5877d878b9-4cmjj               1/1     Running   0          28m
default       myhelmapp-5877d878b9-5vzg9               1/1     Running   0          28m
default       myhelmapp-5877d878b9-mjcq2               1/1     Running   0          28m
default       myhelmapp-5877d878b9-qbw52               1/1     Running   0          28m
default       myhelmapp-5877d878b9-xf6f5               1/1     Running   0          28m
dev           myhelmapp-5877d878b9-4s2dr               1/1     Running   0          47s
dev           myhelmapp-5877d878b9-9lk44               1/1     Running   0          47s
dev           myhelmapp-5877d878b9-gsqfg               1/1     Running   0          47s
dev           myhelmapp-5877d878b9-trj8f               1/1     Running   0          47s
dev           myhelmapp-5877d878b9-xkxd5               1/1     Running   0          47s
prod          myhelmapp-5877d878b9-7lg5l               1/1     Running   0          42s
prod          myhelmapp-5877d878b9-8zzqf               1/1     Running   0          42s
prod          myhelmapp-5877d878b9-gz6qh               1/1     Running   0          42s
prod          myhelmapp-5877d878b9-j4t2x               1/1     Running   0          42s
prod          myhelmapp-5877d878b9-jdfqq               1/1     Running   0          42s
kube-system   coredns-5d78c9869d-c48bf                 1/1     Running   0          3h52m
kube-system   coredns-5d78c9869d-rgh8t                 1/1     Running   0          3h53m
kube-system   etcd-docker-desktop                      1/1     Running   8          49d
kube-system   kube-apiserver-docker-desktop            1/1     Running   8          49d
kube-system   kube-controller-manager-docker-desktop   1/1     Running   8          49d
kube-system   kube-proxy-rhj6w                         1/1     Running   8          49d
kube-system   kube-scheduler-docker-desktop            1/1     Running   28         49d
kube-system   storage-provisioner                      1/1     Running   31         49d
kube-system   vpnkit-controller                        1/1     Running   8          49d
```

```
❯ k get svc -A

NAMESPACE     NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default       myhelmapp    NodePort    10.108.205.89    <none>        80:30860/TCP             144m
dev           myhelmapp    NodePort    10.107.123.144   <none>        80:30621/TCP             5m32s
prod          myhelmapp    NodePort    10.99.30.110     <none>        80:30546/TCP             5m27s
kube-system   kube-dns     ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   49d
default       kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP                  49d
```

```
❯ curl localhost:30860          # default ns

<body style='background-color:#12181b;'>    <h1 style='color:#FFFFFF'>This app was deployed with helm</h1>     <img src='https://raw.githubusercontent.com/devopsjourney1/assets/main/devops-journey-banner.png' alt='CUSTOMER_PHOTO'>    <h2 style='color:#FFFFFF;'>Hello World! Served from <b>myhelmapp-5877d878b9-qbw52</b></h2></body>                                                                                                                                ❯

❯ curl localhost:30621          # dev ns

<body style='background-color:#12181b;'>    <h1 style='color:#FFFFFF'>Deploying in dev, welcome</h1>     <img src='https://raw.githubusercontent.com/devopsjourney1/assets/main/devops-journey-banner.png' alt='CUSTOMER_PHOTO'>    <h2 style='color:#FFFFFF;'>Hello World! Served from <b>myhelmapp-5877d878b9-4s2dr</b></h2></body>                                                                                                                                      

❯ curl localhost:30546         # prod ns

<body style='background-color:#12181b;'>    <h1 style='color:#FFFFFF'>Deploying in production, welcome</h1>     <img src='https://raw.githubusercontent.com/devopsjourney1/assets/main/devops-journey-banner.png' alt='CUSTOMER_PHOTO'>    <h2 style='color:#FFFFFF;'>Hello World! Served from <b>myhelmapp-5877d878b9-gz6qh</b></h2></body>                                                                                                                             
```


```
❯ helm ls -A

NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
mywebapp-release        default         4               2024-07-21 23:08:45.173294 +0545 +0545  deployed        webapp1-0.1.0   1.16.0
mywebapp-release        dev             1               2024-07-21 23:23:36.359399 +0545 +0545  deployed        webapp1-0.1.0   1.16.0
mywebapp-release        prod            1               2024-07-21 23:23:41.234732 +0545 +0545  deployed        webapp1-0.1.0   1.16.0
```
