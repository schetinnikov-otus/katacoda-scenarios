Созданный сервис имеет тип ClusterIP, это значит, что нашему сервису был назначен виртуальный IP. Узнать его мы можем из статуса сервиса. 

Также можно получить полную информацию об объекте Service в формате json. Иногда это бывает полезно для баш скриптов.
`kubectl get service hello-service -o json | jq`{{execute T1}}

`CLUSTER_IP=$(kubectl get service hello-service -o jsonpath="{.spec.clusterIP}")`{{execute T1}}

`echo $CLUSTER_IP`{{execute T1}}

controlplane $ echo $CLUSTER_IP
10.108.237.251

Давайте обратимся к сервису:

`curl http://$CLUSTER_IP:9000/`{{execute T1}}

curl http://$CLUSTER_IP:9000/
Hello world from hello-deployment-d67cff5cc-c7hpw!

Если запустим в цикле, то будем получать ответы от разных подов:

`while true; do curl http://$CLUSTER_IP:9000/ ; echo ''; sleep .5; done`{{execute T1}}

$ while true; do curl http://$CLUSTER_IP:9000/ ; echo ''; sleep .5; done
Hello world from hello-deployment-d67cff5cc-c47w5!
Hello world from hello-deployment-d67cff5cc-q6xcw!
Hello world from hello-deployment-d67cff5cc-c7hpw!
Hello world from hello-deployment-d67cff5cc-c7hpw!
Hello world from hello-deployment-d67cff5cc-q6xcw!
Hello world from hello-deployment-d67cff5cc-c47w5!
Hello world from hello-deployment-d67cff5cc-c7hpw!
Hello world from hello-deployment-d67cff5cc-c7hpw!
Hello world from hello-deployment-d67cff5cc-q6xcw!
Hello world from hello-deployment-d67cff5cc-q6xcw!
Hello world from hello-deployment-d67cff5cc-q6xcw!
Hello world from hello-deployment-d67cff5cc-c47w5!
Hello world from hello-deployment-d67cff5cc-c47w5!
Hello world from hello-deployment-d67cff5cc-q6xcw!
^C
controlplane $ 
