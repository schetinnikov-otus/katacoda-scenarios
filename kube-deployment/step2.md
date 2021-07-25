## Масштабирование деплоймента

Давайте поменяем количество реплик в манифесте

<pre class="file" data-filename="./deployment.yaml" data-target="insert" data-marker="  replicas: 2">
  replicas: 3</pre>
  
И применим его.

`kubectl apply -f deployment.yaml`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как создаcтся еще одна пода.

## Масштабирование деплоймента с помощью kubectl scale 

Также мы можем масштабировать деплоймент с помощью императивной команды kubectl scale.

Например, 

`kubectl scale deploy/hello-deployment --replicas=2`{{execute T1}}

Во второй вкладке можем наблюдать за тем, как сначала удаляется пода:

```
NAME                               READY   STATUS        RESTARTS   AGE
hello-deployment-d67cff5cc-f96r6   1/1     Terminating   0          16s
hello-deployment-d67cff5cc-hrfh8   1/1     Running       0          95s
hello-deployment-d67cff5cc-hsf6g   1/1     Running       0          95s
```

А после команды:

`kubectl scale deploy/hello-deployment --replicas=3`{{execute T1}}

Возвращается создается еще один под:
```
NAME                               READY   STATUS    RESTARTS   AGE
hello-deployment-d67cff5cc-8zbpd   1/1     Running   0          4s
hello-deployment-d67cff5cc-hrfh8   1/1     Running   0          2m55s
hello-deployment-d67cff5cc-hsf6g   1/1     Running   0          2m55s
```
