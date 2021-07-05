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
          image: schetinnikov/hello-app:v3
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

`kubectl apply -f deployment.yaml -f service.yaml`{{execute T1}}


`CLUSTER_IP=$(kubectl get service hello-service -o jsonpath="{.spec.clusterIP}")`{{execute T1}}

`curl -s http://$CLUSTER_IP:9000/env | jq`{{execute T1}}


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
          image: schetinnikov/hello-app:v3
          ports:
            - containerPort: 8000
          env:
            - name: DATABASE_URI
              value: 'postgresql+psycopg2://myuser:passwd@postgres.default.svc.cluster.local:5432/myapp'
            - name: GREETING
              value: 'Alloha'
</pre>

`kubectl apply -f deployment.yaml`{{execute T1}}

`curl -s http://$CLUSTER_IP:9000/env | jq`{{execute T1}}


<pre class="file" data-filename="./config.yaml" data-target="replace">
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-config
data:
  GREETING: Privet

---
apiVersion: v1
kind: Secret
metadata:
  name: hello-secret
data:
  DATABASE_URI: cG9zdGdyZXNxbCtwc3ljb3BnMjovL215dXNlcjpwYXNzd2RAcG9zdGdyZXMvbXlhcHAK
</pre>

`echo 'cG9zdGdyZXNxbCtwc3ljb3BnMjovL215dXNlcjpwYXNzd2RAcG9zdGdyZXMvbXlhcHAK' | base64 -d`{{execute T1}}

`kubectl apply -f config.yaml`{{execute T1}}

`kubectl get cm hello-config`{{execute T1}}

`kubectl get secret hello-secret`{{execute T1}}



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
          image: schetinnikov/hello-app:v3
          ports:
            - containerPort: 8000
          env:
            - name: DATABASE_URI
              valueFrom:
                secretKeyRef:
                  name: hello-secret
                  key: DATABASE_URI
            - name: GREETING
              valueFrom:
                configMapKeyRef:
                  name: hello-config
                  key: GREETING
</pre>

`kubectl apply -f deployment.yaml`{{execute T1}}

`curl -s http://$CLUSTER_IP:9000/env | jq`{{execute T1}}





<pre class="file" data-filename="./special-config.yaml" data-target="replace">
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
data:
  test.json: |
    {
       "status": "OK"
    }
  my.cfg: |
    foo=bar
    baz=quux
  .env: |
    DATABASE_URI=....
</pre>


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
          image: schetinnikov/hello-app:v3
          ports:
            - containerPort: 8000
          volumeMounts:
            - name: special-config
              mountPath: /tmp/config
      volumes:
        - name: special-config
          configMap:
            name: special-config
</pre>

`kubectl apply -f deployment.yaml`{{execute T1}}


`kubectl exec -it deployments/hello-deployment -- /bin/bash`{{execute T3}}

`ls /tmp/config`{{execute T3}}


<pre class="file" data-filename="./special-config.yaml" data-target="insert" data-marker="    foo=bar">
    foo=FOOBARBAZQUUX</pre>

`kubectl apply -f special-config.yaml`{{execute T1}}