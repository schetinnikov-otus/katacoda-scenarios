Теперь давайте посмотрим на тип LoadBalancer. Поскольку контроллера, который бы обрабатывал этот тип сервисов нет, то мы должны увидеть статус Pending в external ip

Правим манифест: 

<pre class="file" data-filename="./service.yaml" data-target="insert" data-marker="  type: NodePort">
  type: LoadBalancer</pre>

Применяем его 

`kubectl apply -f service.yaml`{{execute T1}}

Смотрим, что получилось 

`kubectl describe service hello-service`{{execute T1}}

NAME                      TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/hello-service     LoadBalancer   10.108.237.251   172.17.0.15   9000:32296/TCP   6m53s

`kubectl get service hello-service -o jsonpath="{.spec.ports[0].nodePort}"`{{execute T1}}

`EXTERNAL_IP=$(kubectl get service hello-service -o jsonpath="{.status.loadBalancer.ingress[0].ip}")`{{execute T1}}

Давайте обратимся к сервису:

`curl http://$EXTERNAL_IP:9000/`{{execute T1}}

curl http://$EXTERNAL_IP:9000/
Hello world from hello-deployment-d67cff5cc-c47w5!

Если запустим в цикле, то будем получать ответы от разных подов:

`while true; do curl http://$EXTERNAL_IP:9000/ ; echo ''; sleep .5; done`{{execute T1}}

while true; do curl http://$EXTERNAL_IP:9000/ ; echo ''; sleep .5; done
Hello world from hello-deployment-d67cff5cc-c7hpw!
Hello world from hello-deployment-d67cff5cc-c7hpw!
Hello world from hello-deployment-d67cff5cc-c47w5!
Hello world from hello-deployment-d67cff5cc-c47w5!
Hello world from hello-deployment-d67cff5cc-c7hpw!
Hello world from hello-deployment-d67cff5cc-c47w5!
Hello world from hello-deployment-d67cff5cc-c47w5!
Hello world from hello-deployment-d67cff5cc-c7hpw!
^C
controlplane $
