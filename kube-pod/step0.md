Для того, чтобы запустить кластер Kubernetes надо выполнить: 

`launch.sh`{{execute}}

Как только команда отработает, можно проверить статус нод кластера:

`kubectl get nodes`{{execute}}

и можем посмотреть список неймспейсов 

`kubectl get namespace`{{execute}}

`kubectl create namespace myapp`{{execute}}

`kubectl get pods -A`{{execute}}

`kubectl config set-context --current --namespace=myapp`{{execute}}


Создадим под, для этого запустим в первом терминале:
`kubectl apply -f pod.yaml`{{execute T1}}

`kubectl get pods`{{execute T1}}

`kubectl logs hello-demo`{{execute T1}}

`kubectl exec -it hello-demo -- /bin/bash`{{execute T1}}

Сначала будет только одна (мастер) нода, через некоторое время добавится вторая (рабочая). 

Давайте зайдем на рабочую ноду, для этого откроем новый терминал 

`ssh node01`{{execute T2}}

Запускаем на мастер ноде: 
`docker ps | grep hello`{{execute T1}}

Запускаем на рабочей ноде:
`docker ps | grep hello`{{execute T2}}

Должны увидеть два контейнера.

Получить доступ к поду можно по ip. 
Для этого, вытаскиваем ip командой describe 

`kubectl describe pod hello-demo`{{execute T1}}

`kubectl get -o json pod hello-demo`{{execute}}

`kubectl get -o jsonpath='{.status.podIP}' pod hello-demo`{{execute}}

`POD_IP=$(kubectl get -o jsonpath='{.status.podIP}' pod hello-demo)`{{execute}}

`curl http://$POD_IP:8000/`{{execute}}

`kubectl port-forward hello-demo 9000:8000`{{execute}}

Откроем и увидим

https://[[​HOST_SUBDOMAIN]]-9000-[[KATACODA_HOST]]


Удалим под:
`kubectl delete -f pod.yaml`{{execute T1}}

За процессом удаления можно смотреть в третий терминал. 

Удалятся под может достаточно долго (до минуты)
