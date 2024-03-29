### Создание нового пространства имен

Давайте с вами создадим новое пространство имен. Для изменения объектов, мы можем использовать команды:

- **kubectl create**  для создания ресурсов
- **kubectl edit** / **kubectl patch** для редактирования ресурсов
- **kubectl delete** для удаления ресурсов

Создадим новое пространство имен `myapp`:

`kubectl create ns myapp`{{execute T1}}

Убедимся, что новый **namespace** появился:

`kubectl get ns`{{execute T1}}

И с помощью команды `kubectl config` установим пространство имен по умолчанию в **myapp**.

`kubectl config set-context --current --namespace=myapp`{{execute}}

### Запуск рабочей нагрузки

Давайте контейнер, который мы с вами запускали до этого, теперь запустим с помощью команды `kubectl create`. Для изменения и создания *объектов*, используют команды **kubectl**, которые берут на вход описание *объектов*:

- `kubectl create -f {файл с описанием объекта}`  для создания объектов
- `kubectl apply -f {файл с описанием объекта}`  для обновления объектов
- `kubectl delete -f {файл с описанием объекта}`  для удаления объекта

Описание *объекта* может быть в различных форматах - **json**, **yaml** и т.д.

Внутри **kubectl** на основе текущего пространства имен и по стандартным для объекта полям - **apiVersion**, **kind**, **metadata.name**, находит соответствующий ресурс в API и совершает запросы к **API Server**-у. 

Мы можем воспользоваться описанием ресурса в файле **hello-service.json** и создать объект типа **Pod** с помощью команды:

`kubectl create -f hello-service.json`{{execute T1}}

Запрос, который сделает **kubectl** в **API Server**, будет идентичен запросу, который мы с вами до этого делали напрямую. 

Теперь можем посмотреть на статус объекта типа **Pod**:

`kubectl get pod hello-demo`{{execute T1}}

И удалить ресурс, отвечающий за рабочую нагрузку с помощью команды

`kubectl delete -f hello-service.json`{{execute T1}}

Это будет равносильно тому запросу, который мы делали до этого в **API Server**.

`clear`{{execute T1}} `clear`{{execute T2}}