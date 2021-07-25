## Запуск кластера Kubernetes и создание namespace
Сначала запустим кластер **Kubernetes**. Для этого нужно дождаться выполнения команды:

`./launch_k8s.sh`{{execute}}

Давайте с вами создадим свой **namespace**, в котором будем работать:

`kubectl create namespace myapp`{{execute}}

Чтобы каждый раз не вводить название **namespace**-а в командах **kubectl** изменим контекст:

`kubectl config set-context --current --namespace=myapp`{{execute}}

Для того, чтобы наблюдать тем, как создаются и убиваются поды, запустим во втором терминале команду:

`watch kubectl get pods`{{execute T2}}

> Если терминал не был до этого открыт, то команду нужно будет нажать 2 раза - первый раз будет открыт терминал, а во второй выполнится уже команда

## Создание деплоймента

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

Применим манифест

`kubectl apply -f deployment.yaml`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как создаются поды. 
Дождемся, пока деплоймент раскатится - т.е. когда все поды станут в статусе **Running**

## Масштабирование деплоймента

Давайте поменяем количество реплик в манифесте

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="  replicas: 2">
  replicas: 3</pre>
  
И применим его.

`kubectl apply -f deployment.yaml`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как создаcтся еще одна пода.

## Масштабирование деплоймента с помощью kubectl scale 

Также мы можем масштабировать деплоймент с помощью императивной команды kubectl scale.

Например, 

`kubectl scale deploy/hello-deployment --replicas=2`{{execute T1}}

и возвращаем обратно:

`kubectl scale deploy/hello-deployment --replicas=3`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как сначала удаляется пода, а потом опять создается.


## Удаление поды деплоймента 

Давайте удалим один из подов деплоймента, и посмотрим, что произойдет. 

Чтобы получить список всех под деплоймента, давайте воспользуемся параметром -l в команде kubectl get. Этот параметр позволяет фильтровать все объекты, имеющие соответствующие метки. Так мы отфильтруем все поды деплоймента:

`kubectl get pod -l app=hello-demo`{{execute T1}}

А теперь с помощью jsonpath выведем имя первого пода из списка:

`kubectl get pod -l app=hello-demo -o jsonpath="{.items[0].metadata.name}"`{{execute T1}}

Запомним его в переменную `POD_NAME`

`POD_NAME=$(kubectl get pod -l app=hello-demo -o jsonpath="{.items[0].metadata.name}")`{{execute T1}}

И удалим:

`kubectl delete pod $POD_NAME`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как создаcтся еще одна новая пода.

## Удаление метки с пода деплоймента

Теперь попробуем убрать метку с пода деплоймента, а потом его вернуть обратно:

Запомним в переменную POD_NAME имя первого пода в деплойменте: 

`POD_NAME=$(kubectl get pod -l app=hello-demo -o jsonpath="{.items[0].metadata.name}")`{{execute T1}}

С помощью команды kubectl label мы можем динамически добавлять, убирать и менять метки объектов. 

Удаляем метку **app** с первого пода:

`kubectl label pod $POD_NAME app-`{{execute T1}}

Проверяем, что метку действительно убрали: 

`kubectl get pods --show-labels`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как под без лейбла запущен, а деплоймент создал еще одну новую поду.

А если мы вернем поду его метку:

`kubectl label pod $POD_NAME app=hello-demo`{{execute T1}}

Во второй вкладке можем наблюдать за тем, деплоймент удалил один из подов.

## Стратегия обновления RollingUpdate

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

## Обновление деплоймента с помощью kubectl set image и kubectl rollout undo

Мы также можем обновить версию деплоймента и откатить его с помощью императивных команд kubectl. 

Для обновления на новую версию:

`kubectl set image deploy/hello-deployment hello-demo=schetinnikov/hello-app:v2`{{execute T1}}

А чтобы откатить:

`kubectl rollout undo deploy/hello-deployment`{{execute T1}}

## Стратегия обновления Recreate

Теперь посмотрим, как работает стратегия **Recreate**

Правим манифест

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="    type: RollingUpdate">
    type: Recreate</pre>

и обновляем версию 

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="          image: schetinnikov/hello-app:v1">
          image: schetinnikov/hello-app:v2</pre>

Применяем манифест. 

`kubectl apply -f deployment.yaml`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как одновременно сначала все поды находятся в статусе **Terminating** и после их завершения, создаются новые.

## Удаление деплоймента

Теперь можем удалить деплоймент:

`kubectl delete -f deployment.yaml`{{execute T1}}
