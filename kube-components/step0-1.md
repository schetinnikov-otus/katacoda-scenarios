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

`clear`{{execute T1}}
`docker ps | grep -v pause`{{execute T1}}

Поскольку эта инсталляция **Kubernetes** не является минимальной, помимо базовых компонентов, про которые мы с вами говорили, тут присутствую и другие, дополнительные компоненты. Отфильтруем только компоненты управляющего слоя **etcd**, **kube-scheduler**, **kube-controller-manager** и **API Server**, а также агент **kube-proxy**:

`clear`{{execute T1}}
`docker ps | grep -v pause | grep -E 'etcd|apiserver|scheduler|kube-controller-manager|kube-proxy'`{{execute T1}}

на рабочей ноде

`clear`{{execute T2}}
`docker ps | grep -v pause | grep -E 'etcd|apiserver|scheduler|kube-controller-manager|kube-proxy'`{{execute T2}}

Как видим, на рабочей ноде нет управлящих компонентов, таких как **etcd**, **kube-scheduler**, **kube-controller-manager** и **API Server**, только агент **kube-proxy**.

Давайте посмотрим, как компоненты работают. 

`clear`{{execute T1}} `clear`{{execute T2}}
