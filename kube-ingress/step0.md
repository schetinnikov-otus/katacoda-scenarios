`launch.sh`{{execute}}

`watch kubectl get nodes`{{execute}}

`kubectl create namespace myapp`{{execute}}

`kubectl config set-context --current --namespace=myapp`{{execute}}

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

<pre class="file" data-filename="./ingress.yaml" data-target="replace">
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - http:
      paths:
        - path: /myapp($|/)(.*)
          backend:
            serviceName: hello-service
            servicePort: 9000
</pre>

`kubectl apply -f deployment.yaml -f service.yaml -f ingress.yaml`{{execute T1}}


```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx ingress-nginx/ingress-nginx
```{{execute T1}}


`NGINX_CLUSTER_IP=$(kubectl get service nginx-ingress-nginx-controller -o jsonpath="{.spec.clusterIP}")`{{execute T1}}

`curl $NGINX_CLUSTER_IP/myapp/`{{execute T1}}

`curl $NGINX_CLUSTER_IP/myapp/version`{{execute T1}}
