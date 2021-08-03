## Установка ингресс контроллера

Поскольку у нас нет **ингресс-контроллера** встроенного, его необходимо поставить. 

Ставить будем в системный **namespace** `kube-system` с помощью утилиты **helm**:

`helm repo add bitnami https://charts.bitnami.com/bitnami`{{execute T1}}

`helm install nginx bitnami/nginx-ingress-controller -n kube-system`{{execute T1}}

**Ингресс-контроллер** в данном случае - это **nginx** и **контроллер**, который читает изменения сущности **Ingress**. **Nginx** внутри **Kubernetes** запущен, как обычное приложение, и для него также есть **Service**. Тип сервиса LoadBalancer-а, т.е. доступ к nginx будет извне кластера по внешнему IP адресу.  

Можно посмотреть на настройки этого сервиса:
`kubectl describe svc nginx-nginx-ingress-controller -n kube-system`{{execute T1}}

```
controlplane $ kubectl describe svc nginx-nginx-ingress-controller -n kube-system
Name:                     nginx-nginx-ingress-controller
Namespace:                kube-system
Labels:                   app.kubernetes.io/component=controller
                          app.kubernetes.io/instance=nginx
                          app.kubernetes.io/managed-by=Helm
                          app.kubernetes.io/name=nginx-ingress-controller
                          helm.sh/chart=nginx-ingress-controller-7.6.17
Annotations:              <none>
Selector:                 app.kubernetes.io/component=controller,app.kubernetes.io/instance=nginx,app.kubernetes.io/name=nginx-ingress-controller
Type:                     LoadBalancer
IP:                       10.104.204.210
LoadBalancer Ingress:     172.17.0.8
Port:                     http  80/TCP
TargetPort:               http/TCP
NodePort:                 http  30325/TCP
Endpoints:                10.244.1.3:80
Port:                     https  443/TCP
TargetPort:               https/TCP
NodePort:                 https  32595/TCP
Endpoints:                10.244.1.3:443
Session Affinity:         None
External Traffic Policy:  Cluster
```


Сохраним значение внешнего **IP** в переменную окружения `NGINX_EXTERNAL_IP`.

`NGINX_EXTERNAL_IP=$(kubectl get service nginx-nginx-ingress-controller -n kube-system -o jsonpath="{.status.loadBalancer.ingress[0].ip}")`{{execute T1}}

Теперь можем сделать запросы с помощью команды **curl**:

`curl $NGINX_EXTERNAL_IP/`{{execute T1}}
```
controlplane $ curl $NGINX_EXTERNAL_IP/
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```



