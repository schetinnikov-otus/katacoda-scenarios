Т.к. для идентификации объектов помимо типа ресурса и имени, еще необходимо пространство имен, то в **kubectl** есть текущий **namespace**, который задается в настройках конфигурации **kubectl**. Если мы хотим работать с объектами из другого **namespace**-a, **kubectl** нужно передать параметр `-n {имя пространства имен}`. По умолчанию используется **namespace** с именем **default**.

В текущем пространстве имен нет объектов типа **pod**:

`kubectl get po`{{execute T1}}

А вот в системном **kube-system** их много:

`kubectl get po -n kube-system`{{execute T1}}

Также можно посмотреть, что объекты типа **Service** также отличаются в разных пространствах имен: 

> За что отвечает объект Service, мы с вами поговорим позже, а пока просто воспринимаем его как еще один из множества объектов и ресурсов Kubernetes

`kubectl get service`{{execute T1}}

`kubectl get service -n kube-system`{{execute T1}}

A команды `kubectl get service -n default`{{execute T1}} и `kubectl get service`{{execute T1}} идентичны, потому что в конфиге **kubectl** пространством имен по умолчанию является **default**.

Давайте с вами создадим пространство имен и пропишем **namespace** по умолчанию в настройки **kubectl**. До этого мы использовали только команды, с помощью которых мы получали информацию. Для изменения объектов, мы можем использовать команды **kubectl create** - для создания, **kubectl edit** / **kubectl patch** - для редактирования, **kubectl delete** - для удаления.

Создадим новое пространство имен:

`kubectl create ns myapp`{{execute T1}}

Убедимся, что новый объект появился:

`kubectl get ns`{{execute T1}}

И с помощью команды `kubectl config` установим пространство имен по умолчанию в **myapp**.

`kubectl config set-context --current --namespace=myapp`{{execute}}

