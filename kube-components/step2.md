**Kubectl** - это утилита, которая позволяет управлять кластером **Kubernetes**. **kubectl** можно использовать для развертывания приложений, проверки и управления ресурсов кластера, а также для просмотра логов.

**Kubectl** работает как клиент к **API Server**-y, т.е. все команды в конечном итоге превращаются в запросы к **API Server**-y, поэтому необходимо, чтобы **kubectl** был сконфигурирован и имел доступ к **API Server**. 

Проверить, настроен ли **kubectl**, можно с помощью команды: 

`kubectl cluster-info`{{execute T1}}

> Подробнее про установку, настройку и конфигурирование kubectl можно в официальной документации https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/

## Работа с ресурсами и объектами с помощью kubectl

Поскольку конфигурирование и работа с кластером представляет собой работу с *объектами* **Kubernetes**, то **kubectl** заточен под работу с ними. 

Для каждого *объекта* Kubernetes есть соответствующий ресурс **API** в **API Server**-e. Чтобы посмотреть список всех зарегистрированных ресурсов и соответствующих им типов объектов, можно запустить команду

`kubectl api-resources`{{execute T1}}

### Получение списка объектов

Для того, чтобы получить список объектов в табличном виде можно использовать команду `kubectl get {название ресурса}`

Например, список нод:

`kubectl get nodes`{{execute T1}}

Или статус управляющих компонент:

`kubectl get componentstatus`{{execute T1}}

Или список пространств имен:

`kubectl get namespaces`{{execute T1}}

### Получение конкретного объекта

Для того, чтобы получить только один объект в табличном виде можно использовать команду `kubectl get {название ресурса} {имя ресурса}`

Например, `kubectl get node controlplane`{{execute T1}}

Также можно получить не табличное, а полноценное представление в **json** или **yaml** формате, если передать параметр `-o json` или `-o yaml`:

`kubectl get nodes controlplane -o json`{{execute T1}}

`kubectl get nodes controlplane -o yaml`{{execute T1}}

А чтобы получить расширенное, человекочитаемое описание объекта, можно использовать команду `kubectl describe {название ресурса} {имя ресурса}`

Можно запустить и посмотреть человекочитаемое описание для ноды:

`kubectl describe node controlplane`{{execute T1}}

### Сокращения

Название ресурса в командах можно использовать как во множественном числе, так и в единственном. Т.е. команды `kubectl get node` и `kubectl get nodes` идентичны. 

Также можно использовать сокращения. Например, **no** вместо **nodes**, **ns** вместо **namespace** и т.д. 

Например, 

`kubectl get no`{{execute T1}}

`kubectl get ns`{{execute T1}}

`kubectl get no controlplane`{{execute T1}}

Существующие сокращения для ресурсов можно посмотреть в колонке `shortnames` команды `kubectl api-resources` {{execute T1}}. 

Также можно получить несколько ресурсов за раз, перечислив их через запятую, например:

`kubectl get no,ns`{{execute T1}}

### Пространства имен

Часть ресурсов привязываются к пространству имен, а часть - нет.  Те объекты, которые отвечают за глобальные аспекты кластера или, например, сильно связаны с инфраструктурой, как объекты типа **Node** и **Namespace** вынесены из пространств имен. 

Узнать, какие ресурсы привязаны, а какие нет, можно в соответствующий колонке вывода команды `kubectl api-resources`{{execute T1}}.

Например, такие объекты как **Pod**, привязаны к пространству имен. 

Чтобы работать с объектами конкретного **namespace**-a, **kubectl** нужно передать параметр `-n {имя пространства имен}`.

Например, получить список объектов типа **Pod** в пространстве имен `kube-system`: 

`kubectl get pod -n kube-system`{{execute T1}}

А в пространстве имен `default`

 `kubectl get pod -n default`{{execute T1}}

В **kubectl** есть текущий **namespace**, который задается в настройках конфигурации **kubectl**. По умолчанию используется **namespace** с именем `default`. Т.е. команды без параметра `-n` будут использовать **namespace** `default`.

Запустим команду:

`kubectl get pod`{{execute T1}}

и увидим, что ее вывод идентичен предыдущей команде `kubectl get pod -n default`

С помощью команды  `kubectl config set-context --current --namespace={имя namespace}` можно изменить текущее пространство имен, чтобы каждый раз не передавать в параметрах `-n`.

Давайте изменим текущий **namespace** на `kube-system` и посмотрим вывод команды `kubectl get pod`

`kubectl config set-context --current --namespace=kube-system`{{execute T1}}

`kubectl get pod`{{execute T1}}

`clear`{{execute T1}} `clear`{{execute T2}} 
