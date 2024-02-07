# Домашнее задание к занятию «Управление доступом»

### Цель задания

В тестовой среде Kubernetes нужно предоставить ограниченный доступ пользователю.

------

### Чеклист готовности к домашнему заданию

1. Установлено k8s-решение, например MicroK8S.
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым github-репозиторием.

------

### Инструменты / дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) RBAC.
2. [Пользователи и авторизация RBAC в Kubernetes](https://habr.com/ru/company/flant/blog/470503/).
3. [RBAC with Kubernetes in Minikube](https://medium.com/@HoussemDellai/rbac-with-kubernetes-in-minikube-4deed658ea7b).

------

### Задание 1. Создайте конфигурацию для подключения пользователя

1. Создайте и подпишите SSL-сертификат для подключения к кластеру.

Создаем приватный RSA ключ размером 2048 бит и называем его 'sasha'

```shell
openssl genrsa -out sasha.key 2048
```
   
Затем создаем запрос на подпись сертификата для указанного ключа в файл sasha:

```shell
openssl req -new -key sasha.key -out sasha.csr -subj "/CN=sasha/O=main"
```
Создаем самоподписанный сертификат:

```shell
openssl x509 -req -in sasha.csr -CA /var/snap/microk8s/6089/certs/ca.crt -CAkey /var/snap/microk8s/6089/certs/ca.key -CAcreateserial -out sasha.crt -days 500
```

В случаи, если сертификат и ключ создавались на другом сервере, необходимо смигрировать данные файлы на сервер, на котором развернут кластер k8s

2. Настройте конфигурационный файл kubectl для подключения.

Для правки конфигов и создания user=sasha, вводим следующую команду:

```shell
zag1988@k8s-test:~/main/2.4/cert$ kubectl config set-context sasha-context --cluster=microk8s-cluster --user=sasha

Context "sasha-context" created.
```
Проверяем что конфиги изменились:

```shell
zag1988@k8s-test:~/main/2.4/cert$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.128.0.5:16443
  name: microk8s-cluster
contexts:
- context:
    cluster: microk8s-cluster
    user: admin
  name: microk8s
- context:
    cluster: microk8s-cluster
    user: sasha
  name: sasha-context
current-context: microk8s
kind: Config
preferences: {}
users:
- name: admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
- name: sasha
  user:
    client-certificate: cert/sasha.crt
    client-key: cert/sasha.key
```

3. Создайте роли и все необходимые настройки для пользователя.

Перед тем как создавать роли, надо в кластере включить RBAC:

```shell
zag1988@k8s-test:~/.kube$ microk8s enable rbac
Infer repository core for addon rbac
Enabling RBAC
Reconfiguring apiserver
Restarting apiserver
RBAC is enabled
```
```shell
zag1988@k8s-test:~/main/2.4$ kubectl apply -f role.yaml 
role.rbac.authorization.k8s.io/pod-desc-logs created

zag1988@k8s-test:~/main/2.4$ kubectl apply -f roleBinding.yaml 
rolebinding.rbac.authorization.k8s.io/pod-reader created
```

**[Role](main/role.yaml)**

**[RoleBinding](main/roleBinding.yaml)**


4. Предусмотрите права пользователя. Пользователь может просматривать логи подов и их конфигурацию (`kubectl logs pod <pod_id>`, `kubectl describe pod <pod_id>`).

Проверяем, что есть возможность переключиться на пользователя 'sasha' и посмотреть логи запущенных подов ():

```shell
zag1988@k8s-test:~/main/2.4$ kubectl config use-context sasha-context
Switched to context "sasha-context".

zag1988@k8s-test:~/main/2.4$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS      AGE
volume-deployment-664588cb5d-mf78f   1/1     Running   1 (95m ago)   23h

zag1988@k8s-test:~/main/2.4$ kubectl logs volume-deployment-664588cb5d-mf78f
The directory /usr/share/nginx/html is a volume mount.
Therefore, will not over-write index.html
Only logging the container characteristics:
WBITT Network MultiTool (with NGINX) - volume-deployment-664588cb5d-mf78f -  - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)

zag1988@k8s-test:~/main/2.4$ kubectl describe pod volume-deployment-664588cb5d-mf78f
Name:             volume-deployment-664588cb5d-mf78f
Namespace:        default
Priority:         0
Service Account:  default
Node:             k8s-test/10.128.0.5
Start Time:       Tue, 06 Feb 2024 15:59:14 +0000
Labels:           app=main1
                  pod-template-hash=664588cb5d
Annotations:      cni.projectcalico.org/containerID: b456399f9669d84903cd13b3b931d109e9e84d5f802bc1c0b122c78d07de01c0
                  cni.projectcalico.org/podIP: 10.1.137.167/32
                  cni.projectcalico.org/podIPs: 10.1.137.167/32
Status:           Running
IP:               10.1.137.167
IPs:
  IP:           10.1.137.167
Controlled By:  ReplicaSet/volume-deployment-664588cb5d
Containers:
  multitool:
    Container ID:   containerd://25c893158762c24896a9eb324a739fa143e1a1327b6df76f33413a5a924c9ba5
    Image:          wbitt/network-multitool
    Image ID:       docker.io/wbitt/network-multitool@sha256:d1137e87af76ee15cd0b3d4c7e2fcd111ffbd510ccd0af076fc98dddfc50a735
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 07 Feb 2024 13:39:35 +0000
    Last State:     Terminated
      Reason:       Unknown
      Exit Code:    255
      Started:      Tue, 06 Feb 2024 16:00:15 +0000
      Finished:     Wed, 07 Feb 2024 13:38:35 +0000
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html/ from volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-x5cs5 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      index
    Optional:  false
  kube-api-access-x5cs5:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>

```
Удалять поды при этом пользователь 'sasha' не может :

```shell
kubectl delete pods deployment-6f8868855d-2w8mt
Error from server (Forbidden): pods "deployment-6f8868855d-2w8mt" is forbidden: User "sasha" cannot delete resource "pods" in API group "" in the namespace "default"

zag1988@k8s-test:~/main/2.4$ kubectl auth can-i list pods 
yes
zag1988@k8s-test:~/main/2.4$ kubectl auth can-i get pods
yes
zag1988@k8s-test:~/main/2.4$ kubectl auth can-i watch pods
yes
zag1988@k8s-test:~/main/2.4$ kubectl auth can-i delete pods
no

```

5. Предоставьте манифесты и скриншоты и/или вывод необходимых команд.

------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------

