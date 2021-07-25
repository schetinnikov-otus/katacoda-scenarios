Теперь давайте посмотрим на тип LoadBalancer. Поскольку контроллера, который бы обрабатывал этот тип сервисов нет, то мы должны увидеть статус Pending в external ip

Правим манифест: 

<pre class="file" data-filename="./service.yaml" data-target="insert" data-marker="  type: NodePort">
  type: LoadBalancer</pre>

Применяем его 

`kubectl apply -f service.yaml`{{execute T1}}

Смотрим, что получилось 

`kubectl describe service hello-service`{{execute T1}}

