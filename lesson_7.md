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
2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории. 
4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

  ![Описание](https://github.com/MaximovAA/school/blob/main/kub7.jpg)  
  ![Описание](https://github.com/MaximovAA/school/blob/main/kub7-2.jpg)  
  ![Описание](https://github.com/MaximovAA/school/blob/main/kub7-3.jpg)  
  ![Описание](https://github.com/MaximovAA/school/blob/main/kub7-4.jpg)  
```
При удалении Deployment и PVC потеряет статус Bound, но останется существовать в системе
После удаления PV в моем случае файл так же останется существовать, так как его наличие регламентируется политиками:
Reclaim Policy
Current reclaim policies are:
  Retain -- manual reclamation
  Recycle -- basic scrub (rm -rf /thevolume/*)
  Delete -- delete the volume
```
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:
    storage: 10Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /home/ubuntu/manifest/pv1
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-vol
spec:
  #storageClassName: manual
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Mi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: lesson7
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
        persistentVolumeClaim:
          claimName: pvc-vol
```
------

### Задание 2

**Что нужно сделать**

Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Включить и настроить NFS-сервер на MicroK8S.
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

![Описание](https://github.com/MaximovAA/school/blob/main/kub7-5.jpg)  
![Описание](https://github.com/MaximovAA/school/blob/main/kub7-6.jpg)  
![Описание](https://github.com/MaximovAA/school/blob/main/kub7-7.jpg)  

```

```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: lesson7
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
      volumes:
      - name: vol
        persistentVolumeClaim:
          claimName: pvc-vol
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-vol
spec:
  storageClassName: "nfs"
  accessModes:
    - ReadWriteMany
  resources:
    requests:
```
------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
