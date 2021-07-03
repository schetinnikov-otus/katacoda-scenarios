Для того, чтобы запустить кластер Kubernetes надо выполнить: 

`launch.sh`{{execute}}

Как только команда отработает, можно проверить статус нод кластера:

`kubectl get nodes`{{execute}}

`kubectl create namespace myapp`{{execute}}

`kubectl config set-context --current --namespace=myapp`{{execute}}

<pre class="file" data-filename="./deployment.yaml" data-target="replace">
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-demo
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: hello-demo
    spec:
      containers:
        - name: hello-demo
          image: schetinnikov/hello-app:v1
          ports:
            - containerPort: 8000
</pre>

`kubectl apply -f deployment.yaml`{{execute T1}}

`kubectl get pods`{{execute T1}}


<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="  replicas: 2">
  replicas: 3</pre>

`kubectl apply -f deployment.yaml`{{execute T1}}

`watch kubectl get pods`{{execute T2}}


<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="          image: schetinnikov/hello-app:v1">
          image: schetinnikov/hello-app:v2</pre>

`kubectl apply -f deployment.yaml`{{execute T1}}

`kubectl delete -f deployment.yaml`{{execute T1}}

https://[[HOST_SUBDOMAIN]]-9000-[[KATACODA_HOST]].environments.katacoda.com/
