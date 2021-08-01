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

И файл service.yaml с манифестом *сервиса* 

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

Применим манифесты

`kubectl apply -f deployment.yaml -f service.yaml`{{execute T1}}

Во втором терминале можем наблюдать за тем, как создаются *поды*. 
Дождемся, пока *деплоймент* раскатится - т.е. когда все *поды* станут в статусе **Running**

## Обновление на новую версию

Давайте теперь посмотрим, как для внешнего сервиса будет выглядеть процесс выкатки новой версии. 

Для этого перейдем в третий терминал.

В бесконечном цикле начнем опрашивать наш сервис:

`CLUSTER_IP=$(kubectl get service hello-service -o jsonpath="{.spec.clusterIP}")`{{execute T3}}

> Если терминал не был до этого открыт, то команду нужно будет нажать 2 раза - первый раз будет открыт терминал, а во второй выполнится уже команда



`while true; do curl http://$CLUSTER_IP:9000/version ; echo ''; sleep .5; done`{{execute T3}}


Обновляем версию в манифесте:

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="          image: schetinnikov/hello-app:v1">
          image: schetinnikov/hello-app:v2</pre>

Применяем манифест: 

`kubectl apply -f deployment.yaml`{{execute T1}}

Во втором терминале можем наблюдать за тем, как создаются поды. 
А в третьем, как выглядит процесс раскатки. 

И там можно наблюдать ошибки **Connection refused** во время раскатки, после раскатки они пропадают. Так происходит, потому что контейнер стартовал, он жив, но принимать трафик еще не готов. Для того, чтобы такого не было, необходимо настроить механизм проверок. 

## Откат деплоймента

Откатим деплоймент:

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="          image: schetinnikov/hello-app:v2">
          image: schetinnikov/hello-app:v1</pre>

Применим манифест: 

`kubectl apply -f deployment.yaml`{{execute T1}}

Дождемся, пока деплоймент раскатится - т.е. когда все поды станут в статусе **Running**

## Обновление на новую версию с проверками

Давайте добавим в деплойменте **liveness** и **readiness** проверки:

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="          image: schetinnikov/hello-app:v1">
          image: schetinnikov/hello-app:v2</pre>

<pre class="file" data-filename="./deployment.yaml" data-target="append">
          livenessProbe:
            httpGet:
              port: 8000
              path: /
            initialDelaySeconds: 10
            periodSeconds: 5
            timeoutSeconds: 2
          readinessProbe:
            httpGet:
              port: 8000
              path: /health
            initialDelaySeconds: 1
            periodSeconds: 5
            timeoutSeconds: 2
</pre>

Применяем манифест

`kubectl apply -f deployment.yaml`{{execute T1}}

Во втором терминале можем наблюдать за тем, как создаются *поды*. 
А в третьем, как выглядит процесс раскатки. И там можно наблюдать отсутствие ошибок **Connection refused**. При этом также видно, что во время раскатки нам отвечают как старая, так и новая версия приложения. 
