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

И файл **service.yaml** с манифестом сервиса 

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

Дождемся, пока деплоймент раскатится - т.е. когда все поды станут в статусе **Running**

Посмотреть, состояние сервиса можно с помощью команды: 

`kubectl describe service hello-service`{{execute T1}}

Из интересного, можно увидеть список конкретных **ip** подов, на которые будет направляться трафик в поле **Endpoints**.

Обновим количество реплик и убедимся, что сервис подхватит новый под:

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="  replicas: 2">
  replicas: 3</pre>

Применяем манифест:

`kubectl apply -f deployment.yaml`{{execute T1}}

Теперь смотрим в **Endpoints**.

`kubectl describe service hello-service`{{execute T1}}

И он действительно подхватывается.

Поскольку сервис реализуется с помощью правил маршрутизации трафика на iptables, то мы можем посмотреть, какие правила прописаны для нашего сервиса:

`iptables-save | grep hello-service`{{execute T1}}

Мы также могли создать сервис, используя императивную команду **kubectl expose deployment**:

`kubectl expose deployment hello-deployment --type=ClusterIP --name=hello-service-2`{{execute T1}}
