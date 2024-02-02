# Домашнее задание к занятию «Базовые объекты K8S»

### Цель задания

В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Pod с приложением и подключиться к нему со своего локального компьютера. 

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. Описание [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) и примеры манифестов.
2. Описание [Service](https://kubernetes.io/docs/concepts/services-networking/service/).

------

### Задание 1. Создать Pod с именем hello-world

1. Создать манифест (yaml-конфигурацию) Pod.
2. Использовать image - gcr.io/kubernetes-e2e-test-images/echoserver:2.2.
3. Подключиться локально к Pod с помощью `kubectl port-forward` и вывести значение (curl или в браузере).
   ![apply](https://github.com/MaximovAA/school/blob/main/kub2apply.jpg)
   ![helloworld](https://github.com/MaximovAA/school/blob/main/kub2helloworld.jpg)
   ![curl](https://github.com/MaximovAA/school/blob/main/kub2helloworld-curl.jpg)
------

### Задание 2. Создать Service и подключить его к Pod

1. Создать Pod с именем netology-web.
2. Использовать image — gcr.io/kubernetes-e2e-test-images/echoserver:2.2.
3. Создать Service с именем netology-svc и подключить к netology-web.
4. Подключиться локально к Service с помощью `kubectl port-forward` и вывести значение (curl или в браузере).

   ![k9s](https://github.com/MaximovAA/school/blob/main/kub2k9s.jpg)
   ![netology-web](https://github.com/MaximovAA/school/blob/main/kub2netology-web.jpg)
   ![curl](https://github.com/MaximovAA/school/blob/main/kub2netology-webcurl.jpg)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
  labels:
     app: hello-world
spec:
  containers:
  - name: hello-world
    image: gcr.io/kubernetes-e2e-test-images/echoserver:2.2
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: netology-web
  labels:
     app: netology-web
spec:
  containers:
  - name: netology-web
    image: gcr.io/kubernetes-e2e-test-images/echoserver:2.2
    ports:
    - containerPort: 80
```
   
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world
  labels:
    app: hello-world
spec:
  ports:
  - name: web
    port: 80
    protocol: TCP
    targetPort: 80
#   clusterIP: None
  selector:
    app: hello-world
---
apiVersion: v1
kind: Service
metadata:
  name: netology-web
  labels:
    app: netology-web
spec:
  ports:
  - name: web
    port: 80
    protocol: TCP
    targetPort: 80
#  clusterIP: None
  selector:
    app: netology-web
```

  
  ![pods](https://github.com/MaximovAA/school/blob/main/kub2getpods.jpg)
------

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода команд `kubectl get pods`, а также скриншот результата подключения.
3. Репозиторий должен содержать файлы манифестов и ссылки на них в файле README.md.

------

### Критерии оценки
Зачёт — выполнены все задания, ответы даны в развернутой форме, приложены соответствующие скриншоты и файлы проекта, в выполненных заданиях нет противоречий и нарушения логики.

На доработку — задание выполнено частично или не выполнено, в логике выполнения заданий есть противоречия, существенные недостатки.
