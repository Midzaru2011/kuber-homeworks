# Домашнее задание к занятию «Запуск приложений в K8S»

### Цель задания

В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Deployment с приложением, состоящим из нескольких контейнеров, и масштабировать его.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) Init-контейнеров.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.

[Title](main/deployment1.yaml)

2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.

```shell

NAME                        READY   STATUS    RESTARTS   AGE
netology1-bb7bfb8f4-tcpjp   2/2     Running   0          11m

NAME                        READY   STATUS    RESTARTS   AGE
netology1-bb7bfb8f4-tcpjp   2/2     Running   0          12m
netology1-bb7bfb8f4-cbdn6   2/2     Running   0          4s

```
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.

[Title](main/service.yaml)

```shell

NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes          ClusterIP   10.152.183.1     <none>        443/TCP   6d1h
service-for-nginx   ClusterIP   10.152.183.235   <none>        80/TCP    119s

```

<details>
<summary>service for nginx</summary>

![Alt text](<IMG/service for nginx.PNG>)

</details>

5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

```shell
zag1988@k8s-test:~/main$ microk8s kubectl run multitool --image=dergeberl/multitool:latest
pod/multitool created

zag1988@k8s-test:~/main$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
netology1-bb7bfb8f4-tcpjp   2/2     Running   0          27m
netology1-bb7bfb8f4-cbdn6   2/2     Running   0          15m
multitool                   1/1     Running   0          5m22s

zag1988@k8s-test:~/main$ kubectl exec -it multitool -- /bin/bash
The Dockerfile for this container can be found on GitHub:
https://github.com/dergeberl/multitool-container


Feel free to rise an issue or PR if a tool is missing in this container.
root@multitool:/# curl -v 10.152.183.235
*   Trying 10.152.183.235:80...
* Connected to 10.152.183.235 (10.152.183.235) port 80 (#0)
> GET / HTTP/1.1
> Host: 10.152.183.235
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.25.3
< Date: Wed, 24 Jan 2024 16:17:43 GMT
< Content-Type: text/html
< Content-Length: 615
< Last-Modified: Tue, 24 Oct 2023 13:46:47 GMT
< Connection: keep-alive
< ETag: "6537cac7-267"
< Accept-Ranges: bytes
< 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
* Connection #0 to host 10.152.183.235 left intact
```
------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.

[Title](main/deployment_init.yaml)

2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.

```shell

zag1988@k8s-test:~/main$ kubectl logs sasha-pod-5d794c79bc-nhvdn 
Defaulted container "nginx" out of: nginx, init-myservice (init)
Error from server (BadRequest): container "nginx" in pod "sasha-pod-5d794c79bc-nhvdn" is waiting to start: PodInitializing

```

3. Создать и запустить Service. Убедиться, что Init запустился.

```shell
zag1988@k8s-test:~/main$ kubectl get svc
NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes          ClusterIP   10.152.183.1     <none>        443/TCP   6d2h
service-for-nginx   ClusterIP   10.152.183.252   <none>        80/TCP    40s

```

4. Продемонстрировать состояние пода до и после запуска сервиса.


```shell
zag1988@k8s-test:~/main$ kubectl get pods
NAME                         READY   STATUS     RESTARTS   AGE
netology1-bb7bfb8f4-tcpjp    2/2     Running    0          79m
netology1-bb7bfb8f4-cbdn6    2/2     Running    0          66m
multitool                    1/1     Running    0          57m
sasha-pod-5d794c79bc-nhvdn   0/1     Init:0/1   0          3s

zag1988@k8s-test:~/main$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
netology1-bb7bfb8f4-tcpjp    2/2     Running   0          84m
netology1-bb7bfb8f4-cbdn6    2/2     Running   0          72m
multitool                    1/1     Running   0          62m
sasha-pod-5d794c79bc-nhvdn   1/1     Running   0          5m10s

```

------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать файлы манифестов и ссылки на них в файле README.md.

------
