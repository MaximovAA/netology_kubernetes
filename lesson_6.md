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
2. Сделать так, чтобы busybox писал каждые пять секунд в некий файл в общей директории.
3. Обеспечить возможность чтения файла контейнером multitool.
4. Продемонстрировать, что multitool может читать файл, который периодоически обновляется.
5. Предоставить манифесты Deployment в решении, а также скриншоты или вывод команды из п. 4.


  
   ![Описание](https://github.com/MaximovAA/school/blob/main/kub6-pod.jpg)
   ![Описание](https://github.com/MaximovAA/school/blob/main/kub6-cont.jpg)
   ![Описание](https://github.com/MaximovAA/school/blob/main/kub6-catfile.jpg)
   
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: lesson6
  labels:
    app: back
spec:
  replicas: 1
  selector:
    matchLabels:
      app: back
  template:
    metadata:
      labels:
        app: back
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
#        command: ['sh', '-c', 'while true; do cat /input/success.txt; sleep 10; done']
        volumeMounts:
        - name: vol
          mountPath: /input
        env:
          - name: HTTP_PORT
            value: "8080"
        ports:
        - containerPort: 8080
          name: mul
      - name: app1
        image: busybox
        command: ['sh', '-c', 'while true; do echo Success! >> /output/success.txt; sleep 5; done']
        volumeMounts:
        - name: vol
          mountPath: /output
      volumes:
      - name: vol
        hostPath:
          path: /var/data    
```
   
------

### Задание 2

**Что нужно сделать**

Создать DaemonSet приложения, которое может прочитать логи ноды.

1. Создать DaemonSet приложения, состоящего из multitool.
2. Обеспечить возможность чтения файла `/var/log/syslog` кластера MicroK8S.
3. Продемонстрировать возможность чтения файла изнутри пода.
4. Предоставить манифесты Deployment, а также скриншоты или вывод команды из п. 2.


    ![Описание](https://github.com/MaximovAA/school/blob/main/kub6-allpods.jpg)
   ![Описание](https://github.com/MaximovAA/school/blob/main/kub6-syslog.jpg)
   
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: logmachine
  namespace: lesson6
  labels:
    app: back
spec:
  selector:
    matchLabels:
      app: back
  template:
    metadata:
      labels:
        app: back
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
#        command: ['sh', '-c', 'while true; do cat /input/success.txt; sleep 10; done']
        volumeMounts:
        - name: vol
          mountPath: /input
        env:
          - name: HTTP_PORT
            value: "8080"
        ports:
        - containerPort: 8080
          name: mul
      volumes:
      - name: vol
        hostPath:
          path: /var/log
```
   
------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
