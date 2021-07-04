`launch.sh`{{execute}}

`watch kubectl get nodes`{{execute}}

`kubectl create namespace myapp`{{execute}}

`kubectl config set-context --current --namespace=myapp`{{execute}}

donot need?
`watch kubectl get pod,service`{{execute T2}}

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

<pre class="file" data-filename="./service.yaml" data-target="replace">
apiVersion: v1
kind: Service
metadata:
  name: hello-service
spec:
  selector:
    app: hello-demo
  ports:
    - port: 9000
      targetPort: 8000
  type: ClusterIP
</pre>

`CLUSTER_IP=$(kubectl get service hello-service -o jsonpath="{.spec.clusterIP}")`{{execute T2}}

`while true; do curl http://$CLUSTER_IP:9000/version ; echo ''; sleep .5; done`{{execute T2}}


<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="          image: schetinnikov/hello-app:v1">
          image: schetinnikov/hello-app:v2</pre>

`kubectl apply -f deployment.yaml`{{execute T1}}


<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="          image: schetinnikov/hello-app:v2">
          image: schetinnikov/hello-app:v1</pre>

`kubectl apply -f deployment.yaml`{{execute T1}}


<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="          image: schetinnikov/hello-app:v1">
          image: schetinnikov/hello-app:v2</pre>

<pre class="file" data-filename="./deployment.yaml" data-target="append">
           livenessProbe:
              httpGet:
                 port: 8000
                 path: /
              initialDelaySeconds: 10
              periodSeconds: 5
              timeoutSeconds: 2
           readinessProbe:
              httpGet:
                 port: 8000
                 path: /health
              initialDelaySeconds: 1
              periodSeconds: 5
              timeoutSeconds: 2
</pre>

`kubectl apply -f deployment.yaml`{{execute T1}}
