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
          image: schetinnikov/hello-app:v3
          ports:
            - containerPort: 8000
</pre>

Манифест с описанием сервиса 

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

Применим манифесты:

`kubectl apply -f deployment.yaml -f service.yaml`{{execute T1}}

Во втором терминале можем наблюдать за тем, как создаются поды. 
Дождемся, пока деплоймент раскатится - т.е. когда все поды станут в статусе Running

Мы используем версию приложения, которая по ендпоинту /env отдает свою конфигурацию с переменными окружения:

`CLUSTER_IP=$(kubectl get service hello-service -o jsonpath="{.spec.clusterIP}")`{{execute T1}}

По умолчанию приложение отдает следующие настройки:

`curl -s http://$CLUSTER_IP:9000/env | jq`{{execute T1}}

Давайте изменим их, добавим секцию env 

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
          image: schetinnikov/hello-app:v3
          ports:
            - containerPort: 8000
          env:
            - name: DATABASE_URI
              value: 'postgresql+psycopg2://myuser:passwd@postgres.default.svc.cluster.local:5432/myapp'
            - name: GREETING
              value: 'Alloha'
</pre>

Давайте применим манифест

`kubectl apply -f deployment.yaml`{{execute T1}}

Во втором терминале можем наблюдать за тем, как создаются поды. 
Дождемся, пока деплоймент раскатится - т.е. когда все поды станут в статусе Running

Можем увидеть, что после обновления, приложение отдает другие переменные окружения: 

`curl -s http://$CLUSTER_IP:9000/env | jq`{{execute T1}}


Поскольку передавать конфигурацию через env не всегда удобно, давайте передадим конфигурацию через ConfigMap и Secret. 

Создадим манифест:

<pre class="file" data-filename="./config.yaml" data-target="replace">
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-config
data:
  GREETING: Privet

---
apiVersion: v1
kind: Secret
metadata:
  name: hello-secret
data:
  DATABASE_URI: cG9zdGdyZXNxbCtwc3ljb3BnMjovL215dXNlcjpwYXNzd2RAcG9zdGdyZXMvbXlhcHAK
</pre>

Применим его: 

`kubectl apply -f config.yaml`{{execute T1}}

В секрете, как видим информация закодирована с помощью base64:

`echo 'cG9zdGdyZXNxbCtwc3ljb3BnMjovL215dXNlcjpwYXNzd2RAcG9zdGdyZXMvbXlhcHAK' | base64 -d`{{execute T1}}

Получить ConfigMap и Secret можно также как и любой объект:

`kubectl get cm hello-config`{{execute T1}}

`kubectl get secret hello-secret`{{execute T1}}

Теперь пропишем получение переменных окружения из ConfigMap и Secret в деплоймент:

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
          image: schetinnikov/hello-app:v3
          ports:
            - containerPort: 8000
          env:
            - name: DATABASE_URI
              valueFrom:
                secretKeyRef:
                  name: hello-secret
                  key: DATABASE_URI
            - name: GREETING
              valueFrom:
                configMapKeyRef:
                  name: hello-config
                  key: GREETING
</pre>

Применим манифест

`kubectl apply -f deployment.yaml`{{execute T1}}

Во втором терминале можем наблюдать за тем, как создаются поды. 
Дождемся, пока деплоймент раскатится - т.е. когда все поды станут в статусе Running

После раскатки можем увидеть, что env поменялся и конфигурация действительно берется из configMap и Secret.

`curl -s http://$CLUSTER_IP:9000/env | jq`{{execute T1}}


Давайте посмотрим, как можно примонтировать ConfigMap внутрь пода, как файл.

Создадим ConfigMap дополнительно: 

<pre class="file" data-filename="./mountconfig.yaml" data-target="replace">
apiVersion: v1
kind: ConfigMap
metadata:
  name: mount-config
data:
  test.json: |
    {
       "status": "OK"
    }
  my.cfg: |
    foo=bar
    baz=quux
  .env: |
    DATABASE_URI=....
</pre>

Применим манифест:

`kubectl apply -f mountconfig.yaml`{{execute T1}}

И теперь уже монтируем внутрь пода:

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
          image: schetinnikov/hello-app:v3
          ports:
            - containerPort: 8000
          volumeMounts:
            - name: mount-config
              mountPath: /tmp/config
      volumes:
        - name: mount-config
          configMap:
            name: mount-config
</pre>

Применяем манифест:

`kubectl apply -f deployment.yaml`{{execute T1}}

Во втором терминале можем наблюдать за тем, как создаются поды. 
Дождемся, пока деплоймент раскатится - т.е. когда все поды станут в статусе Running

Зайдем в поду деплоймента и посмотрим, как выглядят примонтированные конфиги в третьем терминале:

`kubectl exec -it deployments/hello-deployment -- /bin/bash`{{execute T3}}

(если терминал не был до этого открыт, то команду нужно будет нажать 2 раза - первый раз будет открыт терминал, а во второй выполнится уже команда)

`ls /tmp/config`{{execute T3}}

`cat /tmp/config/test.json`{{execute T3}}

`cat /tmp/config/my.cfg`{{execute T3}}

Если мы с вами изменим ConfigMap, то через некоторое время (в районе 1 минуты), изменения применятся и файлы внутри пода изменятся:

<pre class="file" data-filename="./special-config.yaml" data-target="insert" data-marker="    foo=bar">
    foo=FOOBARBAZQUUX</pre>

Применим манифест:

`kubectl apply -f special-config.yaml`{{execute T1}}

Смотрим изменения:

`cat /tmp/config/my.cfg`{{execute T3}}
