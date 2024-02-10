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
2. Создать Deployment приложения _backend_ из образа multitool. 
3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера. 
4. Продемонстрировать, что приложения видят друг друга с помощью Service.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

![Описание](https://github.com/MaximovAA/school/blob/main/kub5-pods.jpg)  
![Описание](https://github.com/MaximovAA/school/blob/main/kub5-ingresscurl.jpg)  
![Описание](https://github.com/MaximovAA/school/blob/main/kub5-curlmult.jpg)  

  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: lesson5
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
        env:
          - name: HTTP_PORT
            value: "8080"
        ports:
        - containerPort: 8080
          name: mul
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: lesson5
  labels:
    app: front
spec:
  replicas: 3
  selector:
    matchLabels:
      app: front
  template:
    metadata:
      labels:
        app: front
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
          name: ngx
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: front-service
  namespace: lesson5
  labels:
    app: front
spec:
  ports:
  - name: ngx
    port: 9001
    protocol: TCP
    targetPort: 80
    nodePort: 30080
  selector:
    app: front
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  name: back-service
  namespace: lesson5
  labels:
    app: back
spec:
  ports:
  - name: mul
    port: 9002
    protocol: TCP
    targetPort: 8080
    nodePort: 30808
  selector:
    app: back
  type: NodePort
```


------

### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

1. Включить Ingress-controller в MicroK8S.
2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_.
3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
4. Предоставить манифесты и скриншоты или вывод команды п.2.

![Описание](https://github.com/MaximovAA/school/blob/main/kub5-ingress.jpg)    
![Описание](https://github.com/MaximovAA/school/blob/main/kub5-ingresscurl.jpg)  

  
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-front
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: test.exampletestingress.ru
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: front-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: back-service
            port:
              number: 8080
```
------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------
