## Переменные окружения из ConfigMap и Secret

Теперь пропишем получение переменных окружения из **ConfigMap** и **Secret** в *деплоймент*:

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

Во втором терминале можем наблюдать за тем, как создаются *поды*. 
Дождемся, пока деплоймент раскатится - т.е. когда все поды не окажутся в статусе  **Running**

После раскатки можем убедиться, что конфигурация действительно берется из **СonfigMap** и **Secret**.

`curl -s http://$CLUSTER_IP:9000/env | jq`{{execute T1}}



## Монтирование ConfigMap внутрь пода 


Давайте посмотрим, как можно примонтировать **ConfigMap** внутрь пода, как директорию. Каждый файл в директории будет соответствовать одному параметру - имя файла в качества ключа, а содержимое файла - это значение.

Создадим **ConfigMap** из манифеста **mountconfig.yaml**: 

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

Во втором терминале можем наблюдать за тем, как создаются *поды*. 
Дождемся, пока *деплоймент* раскатится - т.е. когда все *поды* станут в статусе **Running**

Зайдем в *поду* *деплоймента* и посмотрим, как выглядят примонтированные конфиги в третьем терминале:

`kubectl exec -it deploy/hello-deployment -- /bin/bash`{{execute T3}}

>  если терминал не был до этого открыт, то команду нужно будет нажать 2 раза - первый раз будет открыт терминал, а во второй выполнится уже команда.

`ls /tmp/config`{{execute T3}}

`cat /tmp/config/test.json`{{execute T3}}

`cat /tmp/config/my.cfg`{{execute T3}}

Если мы с вами изменим **ConfigMap**, то через некоторое время изменения применятся и файлы внутри *пода* изменятся:

<pre class="file" data-filename="./mountconfig.yaml" data-target="insert" data-marker="    foo=bar">
    foo=FOOBARBAZQUUX</pre>


Применим манифест:

`kubectl apply -f mountconfig.yaml`{{execute T1}}

Смотрим изменения:

`cat /tmp/config/my.cfg`{{execute T3}}