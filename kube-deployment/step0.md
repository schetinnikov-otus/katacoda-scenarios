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

`watch kubectl get pods`{{execute T2}}

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

Применим манифест

`kubectl apply -f deployment.yaml`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как создаются поды. 
Дождемся, пока деплоймент раскатится - т.е. когда все поды станут в статусе Running

Давайте поменяем количество реплик в манифесте

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="  replicas: 2">
  replicas: 3</pre>
  
И применим его.

`kubectl apply -f deployment.yaml`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как создаcтся еще одна пода.

Давайте удалим один из подов деплоймента, и посмотрим, что произойдет. 

Для этого сначала найдем имя такого пода: 

`kubectl get pod -l app=hello-demo -o jsonpath="{.items[0].metadata.name}"`{{execute T1}}

Запомним его в переменную POD_NAME

`POD_NAME=$(kubectl get pod -l app=hello-demo -o jsonpath="{.items[0].metadata.name}")`{{execute T1}}

И удалим:

`kubectl delete pod $POD_NAME`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как создаcтся еще одна новая пода.

Теперь попробуем убрать лейбл с пода деплоймента, а потом его вернуть обратно:

Удаляем лейб app с первого пода:

`POD_NAME=$(kubectl get pod -l app=hello-demo -o jsonpath="{.items[0].metadata.name}")`{{execute T1}}

`kubectl label pod $POD_NAME app-`{{execute T1}}

Проверяем, что лейбл действительно убрали: 

`kubectl get pods --show-labels`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как под без лейбла запущен, а деплоймент создал еще одну новую поду.

А если мы вернем поду его лейбл:

`kubectl label pod $POD_NAME app=hello-demo`{{execute T1}}

Во второй вкладке можем наблюдать за тем, деплоймент удалил один из подов.

Теперь давайте посмотрим, как работают стратегии обновления 

Давайте обновим версию в манифесте на v2

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="          image: schetinnikov/hello-app:v1">
          image: schetinnikov/hello-app:v2</pre>

И применим манифест

`kubectl apply -f deployment.yaml`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как одновременно создаются и удаляются поды.

Также мы можем откатить деплоймент. Для этого достаточно вернуть версию назад.

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="          image: schetinnikov/hello-app:v2">
          image: schetinnikov/hello-app:v1</pre>

И применить манифест 

`kubectl apply -f deployment.yaml`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как одновременно создаются и удаляются поды, и деплоймент возвращается на место. Дождемся пока деплоймент полностью откатится. 

Теперь посмотрим, как работает стратегия Recreate

Правим манифест

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="    type: RollingUpdate">
    type: Recreate</pre>

и обновляем версию 

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="          image: schetinnikov/hello-app:v1">
          image: schetinnikov/hello-app:v2</pre>


Применяем манифест. 

`kubectl apply -f deployment.yaml`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как одновременно сначала все поды находятся в статусе Terminating и после их завершения, создаются новые.

Все, теперь можем удалить деплоймент:

`kubectl delete -f deployment.yaml`{{execute T1}}
