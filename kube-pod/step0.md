Сначала запустим кластер Kubernetes. Для этого нужно дождаться выполнения команды:

`launch.sh`{{execute}}

Но и это, к сожалению, еще не все. Также мы должны дождаться пока ноды кластера перейдут в статус Ready. Статус можно посмотреть командой:

`kubectl get nodes`{{execute}}

Как видим, у нас с вами кластер состоит из 2 нод: управляющая нода (controlplane) и рабочая (node01)
Сначала появится только одна (мастер) нода, через некоторое время добавится вторая (рабочая). 

Давайте посмотрим список namespace-ов

`kubectl get namespace`{{execute}}

Видим несколько системных namespace-ов, в частности kube-system, в котором запущены основные управляющие компоненты кластера Kubernetes

`kubectl get pods -n kube-system`{{execute}}

Помимо стандартных управляющих комопнент, можно увидеть сетевой плагин - flannel, компонент для работы с DNS - coredns, и т.д.

Давайте с вами создадим свой namespace, в котором будем работать:

`kubectl create namespace myapp`{{execute}}

Чтобы каждый раз не вводить название namespace-а в командах kubectl изменим контекст:

`kubectl config set-context --current --namespace=myapp`{{execute}}

Давайте создадим свой первый под, для этого воспользуемся простейшим приложением на питоне, у которого есть несколько эндпоинтов, на которые он отвечает -
/ - Пишет Hell from <название хоста>

/version - Отдает версию приложения

/health - Если приложение работает, отвечает {"status": "ok"} 

Для этого создадим файл pod.yaml с манифестом кубернетес

<pre class="file" data-filename="./pod.yaml" data-target="replace">
apiVersion: v1
kind: Pod
metadata:
  name: hello-demo
  labels:
    app: hello-demo
spec:
  containers:
  - name: hello-demo
    image: schetinnikov/hello-app:v1
    ports:
      - containerPort: 8000
</pre>


И теперь создадим под, для этого запустим в первом терминале:

`kubectl apply -f pod.yaml`{{execute T1}}

С помощью команды можем отслеживать статус подов, которые находятся в нашем неймспейсе.

`kubectl get pods`{{execute T1}}

Периодически запуская эту команду, дождемся когда под поднимется и у него будет статус Running


После этого можно посмотреть логи контейнера внутри пода 

`kubectl logs hello-demo`{{execute T1}}

И например, "зайти" внутрь пода и выполнить в контейнере команду в интерактивном режим 

`kubectl exec -it hello-demo -- /bin/bash`{{execute T1}}

Чтобы выйти, надо нажать Ctrl-C

Давайте зайдем на рабочую ноду, для этого откроем новый терминал

`ssh node01`{{execute T2}}

(если терминал не был до этого открыт, то команду нужно будет нажать 2 раза - первый раз будет открыт терминал, а во второй выполнится уже команда)

Запускаем на мастер ноде: 

`docker ps | grep hello`{{execute T1}}

Запускаем на рабочей ноде:

`docker ps | grep hello`{{execute T2}}

Как видим, рабочая нагрузка действительно запущена именно на рабочей ноде (node01)

Получить доступ к поду можно по ip. 

Для этого, вытаскиваем ip командой describe 

`kubectl describe pod hello-demo`{{execute T1}}

Также мы можем получить полностью развернутую информацию о под в json формате 

`kubectl get -o json pod hello-demo | jq`{{execute}}

Также с помощью формата вывода jsonpath можем доставать любую информацию о поде. Это крайне полезно и удобно для работы в скриптах.

`kubectl get -o jsonpath='{.status.podIP}' pod hello-demo`{{execute}}

Давайте определим IP пода в переменную

`POD_IP=$(kubectl get -o jsonpath='{.status.podIP}' pod hello-demo)`{{execute}}

И теперь по этому IP мы можем обратиться к поду

`curl http://$POD_IP:8000/`{{execute}}

Теперь можно удалить под

`kubectl delete -f pod.yaml`{{execute T1}}

Удалятся под может достаточно долго (до минуты)

С помощью команды можем отслеживать статус пода во втором терминале

`kubectl get pods`{{execute T2}}
