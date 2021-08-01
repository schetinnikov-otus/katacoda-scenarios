Поскольку передавать конфигурацию через **env** не всегда удобно, давайте передадим конфигурацию через **ConfigMap** и **Secret**. Для этого нужно эти объекты создать. Это можно сделать несколькими способами.

## Создание ConfigMap из манифеста

Создадим манифест **configmap.yaml**

<pre class="file" data-filename="./configmap.yaml" data-target="replace">
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-config
data:
  GREETING: Privet
</pre>

Применим манифест **configmap.yaml**: 

`kubectl apply -f configmap.yaml`{{execute T1}}

Получить **ConfigMap**  можно также как и любой объект:

`kubectl get cm hello-config`{{execute T1}}

Посмотреть значения параметров ConfigMap можно с помощью команды `kubectl describe`:

`kubectl describe configmap hello-config`{{execute T1}}

## Создание Secret из манифеста

Создадим манифест **secret.yaml**
<pre class="file" data-filename="./secret.yaml" data-target="replace">
apiVersion: v1
kind: Secret
metadata:
  name: hello-secret
data:
  DATABASE_URI: cG9zdGdyZXNxbCtwc3ljb3BnMjovL215dXNlcjpwYXNzd2RAcG9zdGdyZXMubXlhcHAuc3ZjLmNsdXN0ZXIubG9jYWw6NTQzMi9teWFwcA==
</pre>

Насколько мы знаем данные в объекте **ConfigMap** хранятся как есть, а в **Secret**-е кодируются в **base64** . 

Например, значение `DATABASE_URI` закодировано в **base64**.

`echo 'cG9zdGdyZXNxbCtwc3ljb3BnMjovL215dXNlcjpwYXNzd2RAcG9zdGdyZXMubXlhcHAuc3ZjLmNsdXN0ZXIubG9jYWw6NTQzMi9teWFwcA==' | base64 -d`{{execute T1}}

Когда создаем **Secret** с помощью манифестов, то кодировать нужно самостоятельно. Например, с помощью утилиты **base64**: 

`echo -n 'postgresql+psycopg2://myuser:passwd@postgres.myapp.svc.cluster.local:5432/myapp' | base64`{{execute T1}}

Применим манифест **config.yaml**: 

`kubectl apply -f secret.yaml`{{execute T1}}

Получить **Secret** можно также как и любой объект:

`kubectl get secret hello-secret`{{execute T1}}

В **Secret** данные не показываются, но если запросить в формате **yaml** или **json**, то там будет закодированная строка:

`kubectl get secret -o json hello-secret`{{execute T1}}

`kubectl get secret -o yaml hello-secret`{{execute T1}}

Чтобы получить значение конкретного параметра **Secret**-а из командной строки, можно воспользоваться параметром **jsonpath** и **base64**:

`kubectl get secret hello-secret -o jsonpath="{.data.DATABASE_URI}" | base64 -d`{{execute T1}}

## Создание ConfigMap из командной строки

Создать **ConfigMap** и **Secret** можно и с помощью *императивных* команд **Kubernetes**. 

Самый простой способ создать **ConfigMap** или **Secret** из строковых литералов.

**ConfigMap** можно создать с помощью такой команды: 

`kubectl create configmap {имя конфигмапы} --from-literal={ключ1}={значение1} --from-literal={ключ2}={значение2} ...`

Давайте создадим:

`kubectl create configmap hello-config-literal --from-literal=GREETING=Preved --from-literal=GREETING2=ALLOHA`{{execute T1}}

И проверим, что **ConfigMap** создался правильно:

`kubectl describe configmap hello-config-literal`{{execute T1}}

## Создание Secret из командной строки

Для **Secret** чуть по-другому выглядит команда, но очень похоже: `kubectl create secret generic {имя секрета} --from-literal={ключ1}={значение1} --from-literal={ключ2}={значение2} ...`

Давайте создадим **Secret** с `PASSWORD=SuperCoolPassword2`. Данные в команду передаются чистые, незакодированные, а **Kubernetes** сам занимается их кодированием:

`kubectl create secret generic hello-secret-literal --from-literal=PASSWORD=SuperCoolPassword2`{{execute T1}}

Проверим, что данные закодированы:

`kubectl get secret hello-secret-literal -o jsonpath="{.data.PASSWORD}"`{{execute T1}}

И что совпадают с теми данными, что мы отправляли в команде:

`kubectl get secret hello-secret-literal -o jsonpath="{.data.PASSWORD}" | base64 -d`{{execute T1}}

Теперь удалим **Secret** и **ConfigMap**, чтобы они нам не мешали:

`kubectl delete secret hello-secret-literal`{{execute T1}}

`kubectl delete configmap hello-config-literal`{{execute T1}}

## Создание ConfigMap из файлов

Есть еще возможность создавать **ConfigMap** и **Secret** из файлов. В общем случае передается имя директории. И для каждого файла создается пара, где ключом является имя файла, а значением - его содержимое. Для **ConfigMap** данные сохраняются как есть, а данные для **Secret** кодируются в **base64**. 

Давайте с вами создадим директорию `hello-configmap-dir`, а в ней файлы `GREETING` и `GREETING2`:

`mkdir hello-configmap-dir`{{execute T1}}

`echo 'Preved' > hello-configmap-dir/GREETING`{{execute T1}}

`echo 'ALLOHA' > hello-configmap-dir/GREETING2`{{execute T1}}

Теперь с помощью команды `kubectl create configmap {имя конфигмапы} --from-file={путь до директории}`  мы  можем создать **ConfigMap**:

`kubectl create configmap hello-configmap-from-file --from-file=hello-configmap-dir`{{execute T1}}

И проверим, что **ConfigMap** создался правильно:

`kubectl describe configmaps hello-configmap-from-file`{{execute T1}}

## Создание Secret из файлов

Для **Secret**-ов это работает аналогично:

Давайте с вами создадим директорию `hello-secret-dir`, а в ней файл `DATABASE_URI` и `PASSWORD`. В содержимом файла должны хранится данные в незакодированном виде, **Kubernetes** сам их закодирует. 

`mkdir hello-secret-dir`{{execute T1}}

`echo 'postgresql+psycopg2://myuser:passwd@postgres.myapp.svc.cluster.local:5432/myapp' > hello-secret-dir/DATABASE_URI`{{execute T1}}

`echo 'SuperCoolStrongPassword' > hello-secret-dir/PASSWORD`{{execute T1}}

Теперь с помощью команды `kubectl create secret generic {имя секрета} --from-file={путь до директории}`  мы можем создать секрет. 

`kubectl create secret generic hello-secret-from-file --from-file=hello-secret-dir`{{execute T1}}

Проверим, что данные закодированы:

`kubectl get secret hello-secret-from-file -o jsonpath="{.data.PASSWORD}"`{{execute T1}}

`kubectl get secret hello-secret-from-file -o jsonpath="{.data.DATABASE_URI}"`{{execute T1}}

И что совпадают с теми данными, что мы отправляли в команде:

`kubectl get secret hello-secret-from-file -o jsonpath="{.data.PASSWORD}" | base64 -d`{{execute T1}}

Теперь удалим **Secret** и **ConfigMap**, чтобы они нам не мешали:

`kubectl delete secret hello-secret-from-file`{{execute T1}}

`kubectl delete configmap hello-configmap-from-file`{{execute T1}}
