## Запускаем кластер Kubernetes

Сначала запустим кластер **Kubernetes**. Для этого нужно дождаться выполнения команды:

`./launch_k8s.sh`{{execute}}

Кластер у нас небольшой, но настоящий - состоит из 2ух нод - одной управляющей ноды и одной рабочей. Управляющая нода имеет имя хоста - **controlplane**, а рабочая - **node01**. Давайте зайдем на рабочую ноду. 

`ssh node01 `{{execute T2}}

> если терминал не был до этого открыт, то команду нужно будет нажать 2 раза - первый раз будет открыт терминал, а во второй выполнится уже команда
 
## Компоненты Kubernetes
На всех нодах установлено контейнерное окружение. B нашем случае - это **Docker**. А также запущены агенты - **kubelet** и **kube-proxy**. 

Убедиться в этом мы можем, запуском команды `ps -ef | grep /usr/bin/kubelet`, которая выведет все процессы, а потом отфильтрует те, которые содержат команде запуска `/usr/bin/kubelet`.

на управляющей ноде:

`ps -ef | grep /usr/bin/kubelet`{{execute T1}}

на рабочей ноде:

`ps -ef | grep /usr/bin/kubelet`{{execute T2}}

**Kube-proxy** и управляющие компоненты **Kubernetes** запущены в контейнерном окружении и увидеть их можно с помощью команды `docker ps` 

на управляющией ноде

`docker ps`{{execute T1}}

Мы с вами видим, что помимо обычных контейнеров, есть так называемые **pause**-контейнеры, которые используются **Kubernetes** для внутренних нужд, давайте их отфильтруем:

`docker ps | grep -v pause`{{execute T1}}

Поскольку эта инсталляция **Kubernetes** не является минимальной, помимо базовых компонентов, про которые мы с вами говорили, тут присутствую и другие, дополнительные компоненты. Отфильтруем только компоненты управляющего слоя **etcd**, **kube-scheduler**, **kube-controller-manager** и **API Server**, а также агент **kube-proxy**:

`docker ps | grep -v pause | grep -E 'etcd|apiserver|scheduler|kube-controller-manager|kube-proxy'`{{execute T1}}

на рабочей ноде

`docker ps | grep -v pause | grep -E 'etcd|apiserver|scheduler|kube-controller-manager|kube-proxy'`{{execute T2}}

Как видим, на рабочей ноде нет управлящих компонентов, таких как **etcd**, **kube-scheduler**, **kube-controller-manager** и **API Server**, только агент **kube-proxy**.

Давайте посмотрим, как компоненты работают. 

## API Server

Начнем с входной точки для кластера, через которую происходит управление кластером и взаимодействие компонентов - **API Server**

Спроксируем **API Server** на локальный порт `8080` с помощью команды.

`./proxy_api_server_to_localhost_8080.sh &`{{execute T1}}

Теперь обращаясь по локальному порту `8080`, мы можем совершать запросы к **API Server**-у.

Например, мы можем получить конфигурацию и состояние кластера, относящуюся к **controlplane** ноде, сделав запрос:

`curl -s 127.0.0.1:8080/api/v1/nodes/controlplane/ | jq`{{execute T1}}

> jq - это утилита для форматирования, фильтрации и преобразования текстового вывода в формате JSON, мы еще не раз с ней встретимся

В ответ мы получим довольно большой json с информацией о ноде.

Вся эта информация хранится в хранилище **etcd**. Мы можем зайти **etcd** c помощью команды `docker exec` и посмотреть эту конфигурацию. **Etcd** является key-value хранилищем и, зная ключ, можно получить значение с помощью утилиты **etcdctl**. Информация о ноде **controlplane** хранится в ключе `/registry/minions/controlplane`. 

Сохраним id контейнера, в котором запущен **etcd**, в переменную окружения `ETCD_DOCKER_ID`:
`ETCD_DOCKER_ID=$(docker ps | grep -v pause | grep etcd | awk '{print$1}')`{{execute T1}}

И сделаем запрос напрямую в **etcd**:
`docker exec -it $ETCD_DOCKER_ID etcdctl get /registry/minions/controlplane  --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/peer.crt  --key /etc/kubernetes/pki/etcd/peer.key`{{execute T1}}

Также с помощью этой утилиты можем посмотреть, какие еще есть ключи. Их может быть довольно много, поэтому ограничимся 100 записями.

`docker exec -it $ETCD_DOCKER_ID etcdctl get / --prefix --keys-only --limit=100 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/peer.crt  --key /etc/kubernetes/pki/etcd/peer.key`{{execute T1}}

Через **API Server** пользователи кластера (утилиты или человек) и внутренние компоненты кластера получают и обновляют конфигурацию и статус кластера, который хранится, в **etcd**, а также подписываются на изменения. 

## Запустим рабочую нагрузку через API Server

Давайте с вами запустим инстанс сервиса в кластере. Для этого воспользуемся простейшим приложением на питоне, который на / - отдает текст `Hello world from {имя хоста}!`. Образ контейнера хранится на dockerhub - **schetinikov/hello-app:v1**.

Для того, чтобы наблюдать, как работают компоненты, давайте с вами подпишемся на события, относящиеся к нашему экземпляру сервиса с помощью такой команды в соседнем терминале:

`while true; do clear ; curl -s 127.0.0.1:8080/api/v1/events  | jq '.items[] | {message: .message, component: .source.component} | select(.message | index("hello"))' ; sleep 3; done`{{execute T3}}

> если терминал не был до этого открыт, то команду нужно будет нажать 2 раза - первый раз будет открыт терминал, а во второй выполнится уже команда

команда будет висеть, и как только будут появлятся события о нашем сервисе, они здесь будут появлятся. Команда запускает вечный цикл, в котором мы опрашиваем раз 3 секунды **API Server** по урлу `/api/v1/events` и выбираем только события, которые относятся к нашему контейнеру. 

Для того, чтобы создать рабочую нагрузку, нужно сделать запрос к **API Server**-у.
Запрос будет выглядеть следующим образом.

`curl -v -X POST -H "Content-Type: application/json" http://127.0.0.1:8080/api/v1/namespaces/default/pods -d@hello-service.json`{{execute T1}}

Чуть позже мы разберем формат запроса, а сейчас давайте посмотрим, что происходит. Для этого переключимся на третий терминал. В событиях мы с вами можем увидеть события от **scheduler**-а и **kubelet**-a. Дождемся пока не **kubelet** не напишет, что контейнер `hello-demo` был запущен:

В результаты мы должны увидеть последовательность запуска:
```
{
  "message": "Successfully assigned default/hello-demo to node01",
  "component": "default-scheduler"
}
{
  "message": "Pulling image \"schetinnikov/hello-app:v1\"",
  "component": "kubelet"
}
{
  "message": "Successfully pulled image \"schetinnikov/hello-app:v1\"",
  "component": "kubelet"
}
{
  "message": "Created container hello-demo",
  "component": "kubelet"
}
{
  "message": "Started container hello-demo",
  "component": "kubelet"
}
```

После этого также можем с вами увидеть, что на рабочей ноде наш сервис был запущен:

`docker ps | grep -v pause | grep hello`{{execute T2}}

А вот на управлюящей ноде контейнера нет:
`docker ps | grep -v pause | grep hello`{{execute T1}}

После того, как мы посмотрели на основные компоненты кластера **Kubernetes**, давайте обсудим как хранится конфигурация, и что такое объекты **Kubernetes**.
