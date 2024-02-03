# Домашнее задание к занятию «Хранение в K8s. Часть 2»

### Цель задания

В тестовой среде Kubernetes нужно создать PV и продемострировать запись и хранение файлов.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.

------

### Дополнительные материалы для выполнения задания

1. [Инструкция по установке NFS в MicroK8S](https://microk8s.io/docs/nfs). 
2. [Описание Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/). 
3. [Описание динамического провижининга](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/). 
4. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).

------

### Задание 1

**Что нужно сделать**

Создать Deployment приложения, использующего локальный PV, созданный вручную.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.

[Deployment](main/volume-deployment.yaml)

2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.

[PV.yaml](main/pv.yaml)  
![PV](IMG/PV.PNG)


[PVC.yaml](main/pvc.yaml)
![PVC](IMG/PVC.PNG)


3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории. 

```shell
zag1988@k8s-test:~/main/2.2$ kubectl exec volume-deployment-646f458c87-spgvr -c multitool  -- tail -n 10 /sasha/logoutput.txt
Netology!
Netology!
Netology!
Netology!
Netology!
Netology!
Netology!
Netology!
Netology!
Netology!
```

4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.

```shell
zag1988@k8s-test:~/main/2.2$ kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
volume-deployment   2/2     2            2           55m
zag1988@k8s-test:~/main/2.2$ kubectl delete deployments volume-deployment 
deployment.apps "volume-deployment" deleted
zag1988@k8s-test:~/main/2.2$ kubectl get pvc
NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc    Bound    pv       5Gi        RWO                           59m
zag1988@k8s-test:~/main/2.2$ kubectl delete pvc pvc 
persistentvolumeclaim "pvc" deleted
zag1988@k8s-test:~/main/2.2$ kubectl delete pv
error: resource(s) were provided, but no name was specified
zag1988@k8s-test:~/main/2.2$ kubectl get pv
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS   REASON   AGE
pv     5Gi        RWO            Delete           Failed   default/pvc                           47m
zag1988@k8s-test:~/main/2.2$ cd /sasha/
zag1988@k8s-test:/sasha$ ls -l
total 4
drwxr-xr-x 2 root root 4096 Feb  3 08:55 pv
zag1988@k8s-test:/sasha$ cd pv/
zag1988@k8s-test:/sasha/pv$ ls -l
total 12
-rw-r--r-- 1 root root 11000 Feb  3 09:41 logoutput.txt
zag1988@k8s-test:/sasha/pv$ tail -10 logoutput.txt 
Netology!
Netology!
Netology!
Netology!
Netology!
Netology!
Netology!
Netology!
Netology!
Netology!
```
Файл logoutput.txt сохранен локально на ноде кластера k8s. 


5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.

```shell
zag1988@k8s-test:/sasha/pv$ ls -l
total 12
-rw-r--r-- 1 root root 11000 Feb  3 09:41 logoutput.txt

zag1988@k8s-test:/sasha/pv$ kubectl get pv
No resources found
zag1988@k8s-test:/sasha/pv$ ls -l
total 12
-rw-r--r-- 1 root root 11000 Feb  3 09:41 logoutput.txt
```
При монтировании Persistent Volume в директорию Kubernetes, содержимое директории не перезаписывается данными с Persistent Volume, а вместо этого создается ссылка на эти данные. Таким образом, при удалении Persistent Volume директория продолжает ссылаться на существующие данные на удаленном хранилище.

5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------

### Задание 2

**Что нужно сделать**

Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Включить и настроить NFS-сервер на MicroK8S.
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
