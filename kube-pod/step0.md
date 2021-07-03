Для того, чтобы запустить кластер Kubernetes надо выполнить: 

`launch.sh`{{execute}}

Как только команда отработает, можно проверить статус нод кластера:

`kubectl get nodes`{{execute}}

и можем посмотреть список неймспейсов 

`kubectl get namespace`{{execute}}

`kubectl create namespace myapp`{{execute}}

`kubectl get pods -A`{{execute}}

`kubectl config set-context --current --namespace=myapp`{{execute}}

Сначала будет только одна (мастер) нода, через некоторое время добавится вторая (рабочая). 

Давайте зайдем на рабочую ноду, для этого откроем новый терминал 

`ssh node01`{{execute T2}}

И откроем третий терминал и запустим мониторинг ресурсов куба 

`watch kubectl get all`{{execute T3}}

Под

Создадим под, для этого запустим в первом терминале:
`kubectl apply -f pod.yaml`{{execute T1}}

Проверяем статус пода в третьем терминале.

Как только запустится можем проверить, что под реально запустился на рабочей ноде в docker:

Запускаем на мастер ноде: 
`docker ps | grep hello`{{execute T1}}

Не должно быть ничего 

Запускаем на рабочей ноде:
`docker ps | grep hello`{{execute T2}}

Должны увидеть два контейнера.

Получить доступ к поду можно по ip. 
Для этого, вытаскиваем ip командой describe 

`kubectl describe pod hello-app`{{execute T1}}

И можем зайти в него через curl:
`curl http://[[PODID]]:8000/`

Удалим под:
`kubectl delete -f pod.yaml`{{execute T1}}

За процессом удаления можно смотреть в третий терминал. 

Удалятся под может достаточно долго (до минуты)
