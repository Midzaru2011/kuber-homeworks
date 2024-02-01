# Домашнее задание к занятию «Хранение в K8s. Часть 1»

### Цель задания

В тестовой среде Kubernetes нужно обеспечить обмен файлами между контейнерам пода и доступ к логам ноды.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.

------

### Дополнительные материалы для выполнения задания

1. [Инструкция по установке MicroK8S](https://microk8s.io/docs/getting-started).
2. [Описание Volumes](https://kubernetes.io/docs/concepts/storage/volumes/).
3. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).

------

### Задание 1 

**Что нужно сделать**

Создать Deployment приложения, состоящего из двух контейнеров и обменивающихся данными.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.

**[Deployment](main/volume-deployment.yaml)**

2. Сделать так, чтобы busybox писал каждые пять секунд в некий файл в общей директории.
3. Обеспечить возможность чтения файла контейнером multitool.
4. Продемонстрировать, что multitool может читать файл, который периодоически обновляется.

```shell
zag1988@k8s-test:~/main/2.1$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
volume-deployment-7fd7d746df-wv24p   2/2     Running   0          8m43s
volume-deployment-7fd7d746df-h4j8l   2/2     Running   0          8m40s

zag1988@k8s-test:~/main/2.1$ microk8s kubectl exec volume-deployment-7fd7d746df-wv24p -c multitool  -- tail -n 5 /sasha/logoutput.txt
Netology!
Netology!
Netology!
Netology!
Netology!

```
5. Предоставить манифесты Deployment в решении, а также скриншоты или вывод команды из п. 4.
![Tail](IMG/volume.PNG)

------

### Задание 2

**Что нужно сделать**

Создать DaemonSet приложения, которое может прочитать логи ноды.

1. Создать DaemonSet приложения, состоящего из multitool.

**[Daemonset](main/daemonset.yaml)**

2. Обеспечить возможность чтения файла `/var/log/syslog` кластера MicroK8S.

```shell
      volumes:
      - name: k8s
        hostPath:
          path: /var/log/syslog
```

3. Продемонстрировать возможность чтения файла изнутри пода.

![Syslog](<IMG/syslog from daemonset.PNG>)

4. Предоставить манифесты Deployment, а также скриншоты или вывод команды из п. 2.

------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------
