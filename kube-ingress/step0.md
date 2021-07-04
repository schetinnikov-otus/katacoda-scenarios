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
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
spec:
  rules:
  - host: hello.world
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: hello-service
              port:
                number: 9000
</pre>

`kubectl apply -f deployment.yaml -f service.yaml -f ingress.yaml`{{execute T1}}





`kubectl describe service hello-service`{{execute T1}}

`kubectl get service hello-service -o json | jq`{{execute T1}}

`CLUSTER_IP=$(kubectl get service hello-service -o jsonpath="{.spec.clusterIP}")`{{execute T1}}

`echo $CLUSTER_IP`{{execute T1}}

`curl http://$CLUSTER_IP:9000/health`{{execute T1}}

`while true; do curl http://$CLUSTER_IP:9000/ ; echo ''; sleep .5; done`{{execute T1}}

`iptables-save | grep hello-service`{{execute T1}}

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="  replicas: 2">
  replicas: 3</pre>

`kubectl apply -f deployment.yaml`{{execute T1}}

`kubectl describe service hello-service`{{execute T1}}

<pre class="file" data-filename="./service.yaml" data-target="insert" data-marker="  type: ClusterIP">
  type: NodePort</pre>

`kubectl apply -f service.yaml`{{execute T1}}

`kubectl get service hello-service -o json | jq`{{execute T1}}

`kubectl get service hello-service -o jsonpath="{.spec.ports[0].nodePort}"`{{execute T1}}

`NODE_PORT=$(kubectl get service hello-service -o jsonpath="{.spec.ports[0].nodePort}")`{{execute T1}}

`curl http://node01:$NODE_PORT/`{{execute T1}}

`curl http://controlplane:$NODE_PORT/`{{execute T1}}

<pre class="file" data-filename="./service.yaml" data-target="insert" data-marker="  type: NodePort">
  type: LoadBalancer</pre>

`kubectl apply -f service.yaml`{{execute T1}}

`kubectl describe service hello-service`{{execute T1}}


Service discovery

`kubectl run -it --rm busybox --image=busybox`{{execute T1}}

`env | grep HELLO`{{execute T1}}

`wget -qO- http://hello-service:9000/`{{execute T1}}

`wget -qO- http://hello-service.myapp:9000/`{{execute T1}}

`wget -qO- http://hello-service.myapp.svc.cluster.local:9000/`{{execute T1}}

`kubectl delete -f service.yaml -f deployment.yaml`{{execute T1}}
