Обновим манифест сервиса:

<pre class="file" data-filename="./service.yaml" data-target="insert" data-marker="  type: ClusterIP">
  type: NodePort</pre>

Применим его: 

`kubectl apply -f service.yaml`{{execute T1}}

Давайте посмотрим на полные данные сервиса:

`kubectl get service hello-service -o json | jq`{{execute T1}}

Можем оттуда достать NodePort

`kubectl get service hello-service -o jsonpath="{.spec.ports[0].nodePort}"`{{execute T1}}

`NODE_PORT=$(kubectl get service hello-service -o jsonpath="{.spec.ports[0].nodePort}")`{{execute T1}}

При обращении на этот порт на любой ноде кластера Kubernetes, у нас будут происходить обращения к нашему сервису:

`curl http://node01:$NODE_PORT/`{{execute T1}}

`curl http://controlplane:$NODE_PORT/`{{execute T1}}
