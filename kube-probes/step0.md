Сначала запустим кластер Kubernetes. Для этого нужно дождаться выполнения команды:

`launch.sh`{{execute}}

Но и это, к сожалению, еще не все. Также мы должны дождаться пока ноды кластера перейдут в статус Ready. Статус можно посмотреть командой:

`watch kubectl get nodes`{{execute}}

Как видим, у нас с вами кластер состоит из 2 нод: управляющая нода (controlplane) и рабочая (node01)
Сначала появится только одна (мастер) нода, через некоторое время добавится вторая (рабочая). 

Как только статус станет Ready, можно оборвать выполнение команд с помощью Crtl-C

Давайте с вами создадим свой namespace, в котором будем работать:

`kubectl create namespace myapp`{{execute}}

Чтобы каждый раз не вводить название namespace-а в командах kubectl изменим контекст:

`kubectl config set-context --current --namespace=myapp`{{execute}}

Для того, чтобы наблюдать тем, как создаются и убиваются поды, запустим во втором терминале команду:

`watch kubectl get pods,deployments,service`{{execute T2}}

(если терминал не был до этого открыт, то команду нужно будет нажать 2 раза - первый раз будет открыт терминал, а во второй выполнится уже команда)

Создадим deployment.yaml файл с манифестом Kubernetes: 

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

И файл с манифестом сервиса 

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

Во втором терминале можем наблюдать за тем, как создаются поды. 
Дождемся, пока деплоймент раскатится - т.е. когда все поды станут в статусе Running

Давайте теперь посмотрим, как для внешнего сервиса будет выглядеть процесс выкатки новой версии. 

Для этого перейдем в третий терминал.

И в бесконечном цикле начнем опрашивать наш сервис:

`CLUSTER_IP=$(kubectl get service hello-service -o jsonpath="{.spec.clusterIP}")`{{execute T3}}

(если терминал не был до этого открыт, то команду нужно будет нажать 2 раза - первый раз будет открыт терминал, а во второй выполнится уже команда)

`while true; do curl http://$CLUSTER_IP:9000/version ; echo ''; sleep .5; done`{{execute T3}}


Обновляем версию в манифесте:

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="          image: schetinnikov/hello-app:v1">
          image: schetinnikov/hello-app:v2</pre>

Применяем манифест: 

`kubectl apply -f deployment.yaml`{{execute T1}}

Во втором терминале можем наблюдать за тем, как создаются поды. 
А в третьем, как выглядит процесс раскатки. И там можно наблюдать ошибки Connection refused во время раскатки, после раскатки все ок. Это потому что контейнер стартовал, он жив, но принимать трафик еще не готов. Для того, чтобы такого не было, необходимо настроить механизм проверок. 

Откатим деплоймент:

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="          image: schetinnikov/hello-app:v2">
          image: schetinnikov/hello-app:v1</pre>

Применим манифест: 

`kubectl apply -f deployment.yaml`{{execute T1}}

Дождемся, пока деплоймент раскатится - т.е. когда все поды станут в статусе Running

Давайте добавим в деплойменте liveness и readiness проверки:

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

Во втором терминале можем наблюдать за тем, как создаются поды. 
А в третьем, как выглядит процесс раскатки. И там можно наблюдать отсутствие ошибок Connection refused. При этом также видно, что во время раскатки нам отвечают как старая, так и новая версия приложения. 
