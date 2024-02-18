# Домашнее задание к занятию «Конфигурация приложений»

### Цель задания

В тестовой среде Kubernetes необходимо создать конфигурацию и продемонстрировать работу приложения.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8s).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым GitHub-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/configuration/secret/) Secret.
2. [Описание](https://kubernetes.io/docs/concepts/configuration/configmap/) ConfigMap.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment приложения и решить возникшую проблему с помощью ConfigMap. Добавить веб-страницу

1. Создать Deployment приложения, состоящего из контейнеров nginx и multitool.
2. Решить возникшую проблему с помощью ConfigMap.
3. Продемонстрировать, что pod стартовал и оба конейнера работают.
4. Сделать простую веб-страницу и подключить её к Nginx с помощью ConfigMap. Подключить Service и показать вывод curl или в браузере.
5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

![Описание](https://github.com/MaximovAA/school/blob/main/kub8-1.jpg)  
![Описание](https://github.com/MaximovAA/school/blob/main/kub8-2.jpg)  
![Описание](https://github.com/MaximovAA/school/blob/main/kub8-3.jpg)  
![Описание](https://github.com/MaximovAA/school/blob/main/kub8-4.jpg)  

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginxconf
  namespace: lesson8
data:
  index.html: |
    First
      page
        be happy!
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pods-deployment
  namespace: lesson8
  labels:
    app: nginx
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
      - name: nginx
        image: nginx:1.19
        volumeMounts:
        - name: mynginxconf
          mountPath: /usr/share/nginx/html
        ports:
        - containerPort: 80
          name: ngxport
      - name: multitool
        image: wbitt/network-multitool
        envFrom:
        - configMapRef:
            name: multitoolconf
        ports:
        - containerPort: 31080
          name: mulport
      volumes:
      - name: mynginxconf
        configMap:
          name: nginxconf
---
apiVersion: v1
kind: Service
metadata:
  name: ngx-service
  namespace: lesson8
  labels:
    app: nginx
spec:
  ports:
  - name: ngx
    port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30080
  - name: mul
    port: 31080
    protocol: TCP
    targetPort: 31080
  selector:
    app: nginx
  type: NodePort
```
------

### Задание 2. Создать приложение с вашей веб-страницей, доступной по HTTPS 

1. Создать Deployment приложения, состоящего из Nginx.
2. Создать собственную веб-страницу и подключить её как ConfigMap к приложению.
3. Выпустить самоподписной сертификат SSL. Создать Secret для использования сертификата.
4. Создать Ingress и необходимый Service, подключить к нему SSL в вид. Продемонстировать доступ к приложению по HTTPS. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

```
В рамка выполнения использовался самоподписанный сертификат:
```
![Описание](https://github.com/MaximovAA/school/blob/main/kub8-9.jpg)
![Описание](https://github.com/MaximovAA/school/blob/main/kub8-5.jpg)  
![Описание](https://github.com/MaximovAA/school/blob/main/kub8-6.jpg)  
![Описание](https://github.com/MaximovAA/school/blob/main/kub8-7.jpg)  
![Описание](https://github.com/MaximovAA/school/blob/main/kub8-8.jpg)  

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginxconf
  namespace: lesson8
data:
  index.html: |
    First
      page
        be happy!
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginxssl
  namespace: lesson8
data:
  default.conf: |
    server {
    listen       80;
    listen  [::]:80;
    listen 443 ssl;
    server_name  localhost;

    ssl_certificate /etc/nginx/ssl/tls.crt;
    ssl_certificate_key /etc/nginx/ssl/tls.key;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
    }
---
apiVersion: v1
kind: Service
metadata:
  name: ngx-service
  namespace: lesson8
  labels:
    app: nginx
spec:
  ports:
  - name: ngx
    port: 443
    protocol: TCP
    targetPort: 443
    nodePort: 30080
  - name: mul
    port: 31080
    protocol: TCP
    targetPort: 31080
  selector:
    app: nginx
  type: NodePort
---
apiVersion: v1
kind: Secret
metadata:
  name: secret-tls
  namespace: lesson8
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUQzVENDQXNXZ0F3SUJBZ0lVVXYxVy9GQ2lVUVRCSjZxaEF2aXlQZ3FuRW0wd0RRWUpLb1pJaHZjTkFRRUwKQlFBd2ZqRUxNQWtHQTFVRUJoTUNVbFV4RXpBUkJnTlZCQWdNQ2xOdmJXVXRVM1JoZEdVeElUQWZCZ05WQkFvTQpHRWx1ZEdWeWJtVjBJRmRwWkdkcGRITWdVSFI1SUV4MFpERVlNQllHQTFVRUF3d1BNVFU0TGpFMk1DNHhNRGN1Ck1UVXdNUjB3R3dZSktvWklodmNOQVFrQkZnNTBaWE4wUUcxaGFXd3ViV0ZwYkRBZUZ3MHlOREF5TVRneE16VXoKTVRoYUZ3MHlOVEF5TVRjeE16VXpNVGhhTUg0eEN6QUpCZ05WQkFZVEFsSlZNUk13RVFZRFZRUUlEQXBUYjIxbApMVk4wWVhSbE1TRXdId1lEVlFRS0RCaEpiblJsY201bGRDQlhhV1JuYVhSeklGQjBlU0JNZEdReEdEQVdCZ05WCkJBTU1EekUxT0M0eE5qQXVNVEEzTGpFMU1ERWRNQnNHQ1NxR1NJYjNEUUVKQVJZT2RHVnpkRUJ0WVdsc0xtMWgKYVd3d2dnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0SUJEd0F3Z2dFS0FvSUJBUURkOU5UZko4VUNseEdSTy9RbgpZR2F3anM5L1NxbTQyZlJRMHVacmQrUXlKbHl2eHVkcWQrSVJVeHRjK3pjaDZTeVRYZmdHeGFXb0J2ZDdjOTNvCm9DTTlmQ3RySUg1RldCMCs2aU1jU2IvREtUc0ovYVo4TWpEYm1EVG9MS0ZkSVBta0plek9ZV0dQUWF2N2hoWGkKYnJ1ZnZlV2w1QUxGd2dXaXlYNjJHVzNjWi9IS1dtaVl3YXF5S2xVb0JTYjlzaWVlckJHVnRDK1ZibDlPalhWLwpnckFYZkRNdVlSK2o1UEY3QllLdDRlcjJzay9XRlNRUS9FYXhCSkVkMnRkdWF6NDBOcmlrMVVNd1BNNGU5WHU0Ck5qOU1rT3NRL3drKzdRbWl6d0ZWb1RoRVpqVTJFKzlDVUYxUzhrQkNaWmZBZEdwQkR1aU94bmREVVBKZmJDTWEKUDhDUEFnTUJBQUdqVXpCUk1CMEdBMVVkRGdRV0JCUmphajZBc3p6VDE3R1dBMHdCMCtIdjhnWmsrREFmQmdOVgpIU01FR0RBV2dCUmphajZBc3p6VDE3R1dBMHdCMCtIdjhnWmsrREFQQmdOVkhSTUJBZjhFQlRBREFRSC9NQTBHCkNTcUdTSWIzRFFFQkN3VUFBNElCQVFCVDBESko1MlEvZ2E1QzFMSWpJd0VxM0RMLyszZVdBUFpLTHQvWjBQVWQKTzVOakNVN0tBVkg3a1VnMFdVZGRmb2JwRUNScU4xazBBWGNZSnY5T01IeVRkS0lJc3dZYnkvMzdZQXBUWEZMNApyUy9XMnRVRGhwTVZFTmhFZC95cHh3RWtZOHgzazRBVDB6bllrdmwvd3RrcyttTkNTUUM2UVdhbForZzdFSjRWCnJuUEtuNldjd2N2Wjd3QjFjek9Da1BOZml1SGdLTkZWS3c0VmpCUFdabXp1Rm1wZDFQRjN2MzJ3blh0Q1hvYWcKV0gvRCtROFlyOWRwQ0hueWlNcUxhZnRjNUhHS2pTQks1U0tGSUUwU2pQai9hNEJ4SGh1TFFReCtpNHlrVE5BbAozVkVxV0M5YXQzcEZkNzJ1T2lBWmtQVFk3Zk9OSEp6YTdyTER3aXR2NFFrQwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0t
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2Z0lCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktnd2dnU2tBZ0VBQW9JQkFRRGQ5TlRmSjhVQ2x4R1IKTy9RbllHYXdqczkvU3FtNDJmUlEwdVpyZCtReUpseXZ4dWRxZCtJUlV4dGMremNoNlN5VFhmZ0d4YVdvQnZkNwpjOTNvb0NNOWZDdHJJSDVGV0IwKzZpTWNTYi9ES1RzSi9hWjhNakRibURUb0xLRmRJUG1rSmV6T1lXR1BRYXY3CmhoWGlicnVmdmVXbDVBTEZ3Z1dpeVg2MkdXM2NaL0hLV21pWXdhcXlLbFVvQlNiOXNpZWVyQkdWdEMrVmJsOU8KalhWL2dyQVhmRE11WVIrajVQRjdCWUt0NGVyMnNrL1dGU1FRL0VheEJKRWQydGR1YXo0ME5yaWsxVU13UE00ZQo5WHU0Tmo5TWtPc1Evd2srN1FtaXp3RlZvVGhFWmpVMkUrOUNVRjFTOGtCQ1paZkFkR3BCRHVpT3huZERVUEpmCmJDTWFQOENQQWdNQkFBRUNnZ0VBQWF5Q2xRTkhsZnZtUjhZc3BDckdRRlNPaVVHenpLSzNxM09rN215Y2Z5bm0KYWlLRnNiWTRoc2RZWmNrOGpXTEx3ZjJhUndEYmpQYlpOcFhxb0VRQkJkbWZKaFVSY0N0WlpzK0VuZGZnUy91egpNWjI3TGt2OWd5OUsrVVo3YjFyNEFTM1FoblpubXZvK1M2dTZwMTdPeHk3bHpXQzI0MjBYZG1VdSswU1Y4cHRDCk91NkNyRE9Jckt5emNoQ28xcmswaDM3WjB5QVlWZFhObktNSnFZZXU4Slo5UkIzYi9lSmg5bzFqd2dPa2I5VHkKWWdYTnBwNUNmWG1FcHZtVnZyUGNNUU5nUzloRnQzOE14VnpGSDRRL0sxQXBPWFN2ajRqUlpxUU03V0FuSDRndwp6Q3VBdVNjWk4vSXRQZlFZTW0rMkJueHNNcjRpVkRGR1prOVcvVkg5VVFLQmdRRDZZY1BmNjdINmdRdjhWTW8xCjcrcDFGVXhIZFlGMU5kR29pUmhxNjUweTRZZ1hxcFZrbkVZZ243c25tR09QQ09IMURUckdia05QYy9ZSStuOUYKdk5VNkVkT2xXTFo2OWw4UzE4bUZueC94YXB3WmVyWS9FYmZBUUZtZk1sZHZjeE9yWE5DQ1BITThPNU93emtiawpoOTBiWFc2eVQ5cFVVb1lTOHRicEl0NDZpd0tCZ1FEaTc4a1hYQmJNenE3NExsNXl6VDlIY1ExOHNjMnN1UW5WCjcyZkx5RmlFTWpzbzgxZ25GbVo3M1pwMnhDK0ZPdnJ2clFtZlh4YVJLaGVCd3VTNVFXQm9JcnlONkxqTUxQbWcKUVlJQmdIRDhOWjMyTXpJUnV0OGFRMm5sMzF5czBvNkx1OW5FOVFFcHY0REtOdDkyTy85aVNBa3RTcjN3RXQ2UQpCNUxUcWZiR2pRS0JnRlZyQVAzbFZNU1hQZFdKdjZxQy9NT1F6b01hYlYzbFRHOW94ZkhFQzg4TjdmWFU5ajVxCnFlbjdWRWYvendjL0NvY2xTa1hqM0FiQkV0Q2hWTVlmMDhhSnltQ0FVVkRGdUUyZlhGcS9uSkFweExOVWo5UVkKWVUydkptUVBNcEVNKzByYzBTMTlIZnNRZ0NReld6QWZ3YWpTU1M5LzJvWWwyU09od1B1c0w1QmRBb0dCQUkyaAo1YmtraWo1TWlEcndoWlhVcnplaFFTK2ZzS09wNEkrYW1RZEFCSzZNd3d0dHhJaXduRW1XSnI4VTlpdUtnZTV2CjZsK0M4d3lxWG4xbjYyUUxmMlcrdUR0QkVZU3NWU2RGZXlRQXk0TTgyMWM2NEhiY1VEMk44VnU4S1pUYTNJZ3QKTjE2TElxeXhqbW1tRVpVeklOSnY3dnBMZVh1SjYwbXMwR0ExNVlSSkFvR0JBTTlYcXlISFlSTGZxd1VPTk5FUApWdlc5aHoyVEFYSjk2K0pqQkd4MnRrdFV4NzlyeUFZdXB5L2hYdFlWMHRTNjk0TUxhMWI4UTBkQnRKbkRDWVdJCkxpOVlGL0VsTDlNRi95dVVrbERXRVJVd3B6RGl6aEp6Mlk1aXd5Umk2UldhQWhTT0JMOG5aYy9NWTVCN29SUEcKN09ySGF0cDl1SzdpcWZFV2hMUUFTd2F0Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0=
type: kubernetes.io/tls
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  namespace: lesson8
spec:
  rules:
  - host: kubevm.ru-central1.internal
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ngx-service
            port:
              number: 443
  tls:
    - hosts:
      - kubevm.ru-central1.internal
      secretName: secret-tls
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pods-deployment
  namespace: lesson8
  labels:
    app: nginx
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
      - name: nginx
        image: nginx:1.19
        volumeMounts:
        - name: mynginxconf
          mountPath: /usr/share/nginx/html
        - name: mynginxssl
          mountPath: /etc/nginx/conf.d
        - name: secret-volume
          mountPath: /etc/nginx/ssl
        ports:
        - containerPort: 443
          name: ngxport
      volumes:
      - name: mynginxconf
        configMap:
          name: nginxconf
      - name: mynginxssl
        configMap:
          name: nginxssl
      - name: secret-volume
        secret:
          secretName: secret-tls
```
------

### Правила приёма работы

1. Домашняя работа оформляется в своём GitHub-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------
