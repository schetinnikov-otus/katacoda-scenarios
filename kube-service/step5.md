Для демонстрации работы Service discovery нам нужен внешний, по отношению к нашему деплойменту под. Давайте его запустим с помощью команды kubectl run:

`kubectl run -it --rm busybox --image=busybox`{{execute T1}}

Мы находимся "внутри" контейнера, и можем "изнутри" запускать команды. 

Сначала посмотрим, как работает service discovery через переменные окружения:

`env | grep HELLO`{{execute T1}}

/ # env | grep HELLO
HELLO_SERVICE_2_PORT_8000_TCP_ADDR=10.98.145.31
HELLO_SERVICE_2_PORT_8000_TCP_PORT=8000
HELLO_SERVICE_2_PORT_8000_TCP_PROTO=tcp
HELLO_SERVICE_2_SERVICE_HOST=10.98.145.31
HELLO_SERVICE_PORT_9000_TCP_ADDR=10.108.237.251
HELLO_SERVICE_2_PORT_8000_TCP=tcp://10.98.145.31:8000
HELLO_SERVICE_PORT_9000_TCP_PORT=9000
HELLO_SERVICE_2_PORT=tcp://10.98.145.31:8000
HELLO_SERVICE_2_SERVICE_PORT=8000
HELLO_SERVICE_PORT_9000_TCP_PROTO=tcp
HELLO_SERVICE_SERVICE_HOST=10.108.237.251
HELLO_SERVICE_PORT_9000_TCP=tcp://10.108.237.251:9000
HELLO_SERVICE_SERVICE_PORT=9000
HELLO_SERVICE_PORT=tcp://10.108.237.251:9000

А теперь давайте посмотрим, как работает service discovery при обращении по доменным именам:

`wget -qO- http://hello-service:9000/`{{execute T1}}

/ # wget -qO- http://hello-service:9000/
Hello world from hello-deployment-d67cff5cc-c47w5!/ # 

`wget -qO- http://hello-service.myapp:9000/`{{execute T1}}

/ # wget -qO- http://hello-service.myapp:9000/
Hello world from hello-deployment-d67cff5cc-c7hpw!/ # 

`wget -qO- http://hello-service.myapp.svc.cluster.local:9000/`{{execute T1}}
/ # wget -qO- http://hello-service.myapp.svc.cluster.local:9000/
Hello world from hello-deployment-d67cff5cc-c7hpw!/ # 

На этом все, удалить все объекты можно с помощью команды:

`kubectl delete -f service.yaml -f deployment.yaml`{{execute T1}}
