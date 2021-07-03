Для того, чтобы запустить кластер Kubernetes надо выполнить: 

`launch.sh`{{execute}}

Как только команда отработает, можно проверить статус нод кластера:

`kubectl get nodes`{{execute}}

`kubectl create namespace myapp`{{execute}}

`kubectl config set-context --current --namespace=myapp`{{execute}}

`deployment.yaml`{{open}}

`kubectl apply -f deployment.yaml`{{execute T1}}

`kubectl get pods`{{execute T1}}

https://[[HOST_SUBDOMAIN]]-9000-[[KATACODA_HOST]].environments.katacoda.com/

Удалим под:
`kubectl delete -f deployment.yaml`{{execute T1}}
