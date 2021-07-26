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

`clear`{{execute T2}}
`docker ps | grep -v pause | grep hello`{{execute T2}}

А вот на управлюящей ноде контейнера нет:
`clear`{{execute T1}}
`docker ps | grep -v pause | grep hello`{{execute T1}}

После того, как мы посмотрели на основные компоненты кластера **Kubernetes**, давайте обсудим как хранится конфигурация, и что такое объекты **Kubernetes**.

`clear`{{execute T1}} `clear`{{execute T2}}
