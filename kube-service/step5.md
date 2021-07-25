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
