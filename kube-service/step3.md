Обновим манифест сервиса:

<pre class="file" data-filename="./service.yaml" data-target="insert" data-marker="  type: ClusterIP">
  type: NodePort</pre>

Применим его: 

`kubectl apply -f service.yaml`{{execute T1}}

Давайте посмотрим на полные данные сервиса:

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/hello-service     NodePort    10.108.237.251   <none>        9000:32296/TCP   5m12s

`kubectl get service hello-service -o json | jq`{{execute T1}}

Можем оттуда достать NodePort

`kubectl get service hello-service -o jsonpath="{.spec.ports[0].nodePort}"`{{execute T1}}

`NODE_PORT=$(kubectl get service hello-service -o jsonpath="{.spec.ports[0].nodePort}")`{{execute T1}}

При обращении на этот порт на любой ноде кластера Kubernetes, у нас будут происходить обращения к нашему сервису:

`curl http://node01:$NODE_PORT/`{{execute T1}}

curl http://node01:$NODE_PORT/
Hello world from hello-deployment-d67cff5cc-q6xcw!

`curl http://controlplane:$NODE_PORT/`{{execute T1}}

curl http://controlplane:$NODE_PORT/
Hello world from hello-deployment-d67cff5cc-c7hpw!
