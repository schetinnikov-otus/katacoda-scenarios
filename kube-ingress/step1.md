

## Запуск приложения

Создадим **deployment.yaml** файл с манифестом **Kubernetes**: 

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

И файл **service.yaml** с манифестом *сервиса* 

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

И в файле **ingress.yaml** манифест **ингресса**:

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

Давайте применим все манифесты

`kubectl apply -f deployment.yaml -f service.yaml -f ingress.yaml`{{execute T1}}

Во втором терминале можем наблюдать за тем, как создаются *поды*. 
Дождемся, пока *деплоймент* раскатится - т.е. когда все *поды* станут в статусе **Running**

## Установка ингресс контроллера

Поскольку у нас нет **ингресс-контроллера** встроенного, его необходимо поставить. 

Ставить будем в системный **namespace** **kube-system** с помощью утилиты **helm**:

`helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx`{{execute T1}}

`helm install nginx ingress-nginx/ingress-nginx -n kube-system`{{execute T1}}

**Ингресс-контроллер** в данном случае - это **nginx** и **контроллер**, который читает изменения сущности **Ingress**. **Nginx** внутри **Kubernetes** запущен, как обычное приложение, и для него также есть **Service**.  

`kubectl get service nginx-ingress-nginx-controller -o json -n kube-system | jq` {{execute T1}}

`NGINX_CLUSTER_IP=$(kubectl get service nginx-ingress-nginx-controller -o jsonpath="{.spec.clusterIP}")`{{execute T1}}

## Запросы к ингресс-контроллеру

Можно делать запросы к **ингрес-контроллеру** и он будет маршрутизировать трафик в соответствии с правилами из **ингрессов**:

`curl $NGINX_CLUSTER_IP/myapp/`{{execute T1}}

`curl $NGINX_CLUSTER_IP/myapp/version`{{execute T1}}

Также можно обратитьс по внешнему **IP** адресу:

`NGINX_EXTERNAL_IP=$(kubectl get service nginx-ingress-nginx-controller -o jsonpath="{.status.loadBalancer.ingress[0].ip}")`{{execute T1}}

`curl $NGINX_EXTERNAL_IP/myapp/version`{{execute T1}}

`curl $NGINX_EXTERNAL_IP/myapp/`{{execute T1}}
