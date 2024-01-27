# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к приложению, установленному в предыдущем ДЗ и состоящему из двух контейнеров, по разным портам в разные контейнеры как внутри кластера, так и снаружи.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.

[Deployment.yaml](main/deployment.yaml)

2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.

[Service.yaml](main/service.yaml)

3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.

4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.

<details>
<summary> curl -v svc </summary>

```shell
zag1988@k8s-test:~$ kubectl get pods
NAME                          READY   STATUS    RESTARTS       AGE
multitool-599d9cf4cb-nlf29    1/1     Running   1 (163m ago)   43h
deployment-6f8868855d-p2jsf   2/2     Running   2 (163m ago)   43h
deployment-6f8868855d-hjmx8   2/2     Running   2 (163m ago)   43h
deployment-6f8868855d-zv7tg   2/2     Running   2 (163m ago)   43h

zag1988@k8s-test:~$ kubectl exec -it multitool-599d9cf4cb-nlf29 --  /bin/bash
The Dockerfile for this container can be found on GitHub:
https://github.com/dergeberl/multitool-container


Feel free to rise an issue or PR if a tool is missing in this container.
root@multitool-599d9cf4cb-nlf29:/# curl -vv mysvc:9001
*   Trying 10.152.183.177:9001...
* Connected to mysvc (10.152.183.177) port 9001 (#0)
> GET / HTTP/1.1
> Host: mysvc:9001
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.19.1
< Date: Sat, 27 Jan 2024 10:15:29 GMT
< Content-Type: text/html
< Content-Length: 612
< Last-Modified: Tue, 07 Jul 2020 15:52:25 GMT
< Connection: keep-alive
< ETag: "5f049a39-264"
< Accept-Ranges: bytes
< 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
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
* Connection #0 to host mysvc left intact
root@multitool-599d9cf4cb-nlf29:/# curl -vv mysvc:9002
*   Trying 10.152.183.177:9002...
* Connected to mysvc (10.152.183.177) port 9002 (#0)
> GET / HTTP/1.1
> Host: mysvc:9002
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0
< Date: Sat, 27 Jan 2024 10:15:57 GMT
< Content-Type: text/html
< Content-Length: 148
< Last-Modified: Sat, 27 Jan 2024 07:31:44 GMT
< Connection: keep-alive
< ETag: "65b4b160-94"
< Accept-Ranges: bytes
< 
WBITT Network MultiTool (with NGINX) - deployment-6f8868855d-hjmx8 - 10.1.137.134 - HTTP: 8080 , HTTPS: 11443 . (Formerly praqma/network-multitool)
* Connection #0 to host mysvc left intact

```
</details>

5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

------

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.
2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.


<details>
<summary> curl -v NodePort </summary>

```shell
zag1988@mytest-6:~/kuber-homeworks/1.4$ curl -vv 130.193.39.193:32080
*   Trying 130.193.39.193:32080...
* Connected to 130.193.39.193 (130.193.39.193) port 32080 (#0)
> GET / HTTP/1.1
> Host: 130.193.39.193:32080
> User-Agent: curl/7.81.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.19.1
< Date: Sat, 27 Jan 2024 10:27:57 GMT
< Content-Type: text/html
< Content-Length: 612
< Last-Modified: Tue, 07 Jul 2020 15:52:25 GMT
< Connection: keep-alive
< ETag: "5f049a39-264"
< Accept-Ranges: bytes
< 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
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
* Connection #0 to host 130.193.39.193 left intact

zag1988@mytest-6:~/kuber-homeworks/1.4$ curl -vv 130.193.39.193:31280
*   Trying 130.193.39.193:31280...
* Connected to 130.193.39.193 (130.193.39.193) port 31280 (#0)
> GET / HTTP/1.1
> Host: 130.193.39.193:31280
> User-Agent: curl/7.81.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.24.0
< Date: Sat, 27 Jan 2024 10:29:10 GMT
< Content-Type: text/html
< Content-Length: 148
< Last-Modified: Sat, 27 Jan 2024 07:31:44 GMT
< Connection: keep-alive
< ETag: "65b4b160-94"
< Accept-Ranges: bytes
< 
WBITT Network MultiTool (with NGINX) - deployment-6f8868855d-zv7tg - 10.1.137.191 - HTTP: 8080 , HTTPS: 11443 . (Formerly praqma/network-multitool)
* Connection #0 to host 130.193.39.193 left intact

```
</details>

3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.

<details>
<summary> PRTSC NodePort </summary>

![SVC NodePORT.PNG](<main/IMG/SVC NodePORT.PNG>)

</details>



[SVC for NODEport.yaml](main/svc_for_nodPort.yaml)

------

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

