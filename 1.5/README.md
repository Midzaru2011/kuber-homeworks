# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 2»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к двум приложениям снаружи кластера по разным путям.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Service.
3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress.
4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment приложений backend и frontend

1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт.

[Front](main/frontend.yaml)

2. Создать Deployment приложения _backend_ из образа multitool. 

[Back](main/backend,yaml)

3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера. 

[svc-front](main/svc-frontend.yaml)

[svc-back](main/svc-backend.yaml)

4. Продемонстрировать, что приложения видят друг друга с помощью Service.

<details>
<sumarry>Curl to svc</summary>

```shell
zag1988@k8s-test:~/main/1.5$ kubectl get svc
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes         ClusterIP   10.152.183.1     <none>        443/TCP   12d
backend-service    ClusterIP   10.152.183.114   <none>        80/TCP    11s
frontend-service   ClusterIP   10.152.183.172   <none>        80/TCP    6s

zag1988@k8s-test:~/main/1.5$ kubectl exec frontend-d94f45cb8-l66w8 -- curl backend-service
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0WBITT Network MultiTool (with NGINX) - backend-6d5c675b47-hvnkj - 10.1.137.146 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
100   141  100   141    0     0   8294      0 --:--:-- --:--:-- --:--:--  8294


zag1988@k8s-test:~/main/1.5$ kubectl exec frontend-d94f45cb8-l66w8 -- curl frontend-service
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
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
100   612  100   612    0     0   9870      0 --:--:-- --:--:-- --:--:--  9870

```
</details>


5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

------

### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

1. Включить Ingress-controller в MicroK8S.

<details>
<summary>microk8s enable ingress</summary>

```shell
zag1988@k8s-test:~/main/1.5$ microk8s enable ingress
Infer repository core for addon ingress
Enabling Ingress
ingressclass.networking.k8s.io/public created
ingressclass.networking.k8s.io/nginx created
namespace/ingress created
serviceaccount/nginx-ingress-microk8s-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-microk8s-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-microk8s-role created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
configmap/nginx-load-balancer-microk8s-conf created
configmap/nginx-ingress-tcp-microk8s-conf created
configmap/nginx-ingress-udp-microk8s-conf created
daemonset.apps/nginx-ingress-microk8s-controller created
Ingress is enabled

zag1988@k8s-test:~/main/1.5$ kubectl get ns
NAME              STATUS   AGE
kube-system       Active   13d
kube-public       Active   13d
kube-node-lease   Active   13d
default           Active   13d
ingress           Active   2m17s

zag1988@k8s-test:~/main/1.5$ kubectl get pods -n ingress | grep ingress
nginx-ingress-microk8s-controller-wc7f6   1/1     Running   0          2m42s

```
</details>

2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_.


[my-ingress](main/Ingress.yaml)


```shell
zag1988@k8s-test:~/main/1.5/main$ kubectl describe ingress
Name:             my-ingress
Labels:           <none>
Namespace:        default
Address:          127.0.0.1
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /      frontend-service:80 (10.1.137.142:80,10.1.137.143:80,10.1.137.145:80)
              /api   backend-service:80 (10.1.137.146:80)
Annotations:  ingress.kubernetes.io/rewrite-target: /
Events:
  Type    Reason  Age                    From                      Message
  ----    ------  ----                   ----                      -------
  Normal  Sync    2m42s (x2 over 3m41s)  nginx-ingress-controller  Scheduled for sync
```

3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.

![curl localhost](<IMG/curl to Nginx.PNG>)



4. Предоставить манифесты и скриншоты или вывод команды п.2.

------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------
