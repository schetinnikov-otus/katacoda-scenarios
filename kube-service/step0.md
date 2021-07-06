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

Посмотреть, состояние сервиса можно с помощью команды: 

`kubectl describe service hello-service`{{execute T1}}

Из интересного, можно увидеть список конкретных ip подов, на которые будет направляться трафик в поле Endpoints.

Также можно получить полную информацию об объекте Service в формате json. Иногда это бывает полезно для баш скриптов.

`kubectl get service hello-service -o json | jq`{{execute T1}}


Например, ClusterIP сервиса можно записать в переменную:

`CLUSTER_IP=$(kubectl get service hello-service -o jsonpath="{.spec.clusterIP}")`{{execute T1}}

`echo $CLUSTER_IP`{{execute T1}}

Давайте обратимся к сервису:

`curl http://$CLUSTER_IP:9000/`{{execute T1}}

Если запустим в цикле, то будем получать ответы от разных подов:

`while true; do curl http://$CLUSTER_IP:9000/ ; echo ''; sleep .5; done`{{execute T1}}

И мы с вами даже можем посмотреть, какие правила в iptables написаны для нашего сервиса

`iptables-save | grep hello-service`{{execute T1}}


Обновим количество реплик и убедимся, что сервис подхватит новый под:

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="  replicas: 2">
  replicas: 3</pre>

Применяем манифест:

`kubectl apply -f deployment.yaml`{{execute T1}}

Теперь смотрим в Endpoints.

`kubectl describe service hello-service`{{execute T1}}


Давайте посмотрим, как работают другие типы сервисов. Например, NodePort

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

Теперь давайте посмотрим на тип LoadBalancer. Поскольку контроллера, который бы обрабатывал этот тип сервисов нет, то мы должны увидеть статус Pending в external ip

Правим манифест: 

<pre class="file" data-filename="./service.yaml" data-target="insert" data-marker="  type: NodePort">
  type: LoadBalancer</pre>

Применяем его 

`kubectl apply -f service.yaml`{{execute T1}}

Смотрим, что получилось 

`kubectl describe service hello-service`{{execute T1}}


Service discovery

Для демонстрации работы Service discovery нам нужен внешний, по отношению к нашему деплойменту под. Давайте его запустим с помощью команды kubectl run:

`kubectl run -it --rm busybox --image=busybox`{{execute T1}}

Мы находимся "внутри" контейнера, и можем "изнутри" запускать команды. 

Сначала посмотрим, как работает service discovery через переменные окружения:

`env | grep HELLO`{{execute T1}}

А теперь давайте посмотрим, как работает service discovery при обращении по доменным именам:

`wget -qO- http://hello-service:9000/`{{execute T1}}

`wget -qO- http://hello-service.myapp:9000/`{{execute T1}}

`wget -qO- http://hello-service.myapp.svc.cluster.local:9000/`{{execute T1}}


На этом все, удалить все объекты можно с помощью команды:

`kubectl delete -f service.yaml -f deployment.yaml`{{execute T1}}
