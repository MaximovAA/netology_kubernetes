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
2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

  ![Описание](https://github.com/MaximovAA/school/blob/main/kub3-deployment2.jpg)  
  ![Описание](https://github.com/MaximovAA/school/blob/main/kub3-service1.jpg)  
  ![Описание](https://github.com/MaximovAA/school/blob/main/kub3-curlpod.jpg)  
  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pods-deployment
  namespace: default
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
          name: ngxport
      - name: multitool
        image: wbitt/network-multitool
        env:
          - name: HTTP_PORT
            value: "31080"
        ports:
        - containerPort: 31080
          name: mulport
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ngx-service
  labels:
    app: nginx
spec:
  ports:
  - name: ngx
    port: 80
    protocol: TCP
    targetPort: 80
  - name: mul
    port: 31080
    protocol: TCP
    targetPort: 31080
  selector:
    app: nginx
```
  
------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

![Описание](https://github.com/MaximovAA/school/blob/main/kub3-init.jpg)  
![Описание](https://github.com/MaximovAA/school/blob/main/kub3-nslookup.jpg)  
![Описание](https://github.com/MaximovAA/school/blob/main/kub3-initdone.jpg)  
![Описание](https://github.com/MaximovAA/school/blob/main/kuub3-service.jpg)  


```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: init-pod
spec:
  replicas: 1
  selector:
    matchLabels:
        app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.19.2
        name: nginx
        volumeMounts:
        - name: webdata
          mountPath: "/usr/share/nginx/html"
      initContainers:
      - name: init
        image: busybox:1.28
        command: ['sh', '-c', "until nslookup myservice.default.svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
        volumeMounts:
        - name: webdata
          mountPath: "/usr/share/nginx/html"
      volumes:
        - name: webdata
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  selector:
    app: nginx
```

------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать файлы манифестов и ссылки на них в файле README.md.

------
