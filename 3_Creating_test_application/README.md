Создание тестового приложения
Решение:
Подготовим тестовое приложение.

Созададим отдельный git репозиторий Test-application с простым nginx конфигом, который будет отдавать статические данные:

Клонируем репозиторий:

https://github.com/slava1005/Test-application.git

Создадим в этом репозитории файл содержащую HTML-код ниже:
index.html

```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m new DevOps Engineer!</h1>
</body>
</html>
```
Создадим Dockerfile, который будет запускать веб-сервер Nginx в фоне с индекс страницей:
Dockerfile
```
FROM nginx:1.27-alpine

COPY index.html /usr/share/nginx/html
```

Загрузим файлы в Git-репозиторий.

Создадим папку для приложения mkdir mynginx и скопируем в нее ранее созданые файлы.
В этой папке выполним сборку приложения:

sudo docker build -t slava1005/nginx:v1 .
[sudo] password for slava:

DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  12.29kB
Step 1/2 : FROM nginx:1.27-alpine
1.27-alpine: Pulling from library/nginx
1f3e46996e29: Pull complete
5215a08fb124: Pull complete
f8813b38090d: Pull complete
9f41882e104d: Pull complete
e92b9802c411: Pull complete
4b56e0e1b50d: Pull complete
5281c445f8b7: Pull complete
a53100808f89: Pull complete
Digest: sha256:b471bb609adc83f73c2d95148cf1bd683408739a3c09c0afc666ea2af0037aef
Status: Downloaded newer image for nginx:1.27-alpine
 ---> d41a14a4ecff
Step 2/2 : COPY index.html /usr/share/nginx/html
 ---> 4f39d84fd609
Successfully built 4f39d84fd609
Successfully tagged slava1005/nginx:v1

Проверим, что образ создался:

slava@DESKTOP-QKJU13U:~/mynginx$ sudo docker images
REPOSITORY                             TAG           IMAGE ID       CREATED          SIZE
slava1005/nginx                        v1            4f39d84fd609   26 seconds ago   47.9MB

Запустим docker-контейнер с созданным образом и проверим его работоспособность:

slava@DESKTOP-QKJU13U:~/mynginx$ sudo docker run -d -p 80:80 slava1005/nginx:v1
4f191ffc35860e1d50aaaf8246157e1c8d05dc5063d2e72870c23ecef95c1380
slava@DESKTOP-QKJU13U:~/mynginx$ docker ps
CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS          PORTS                               NAMES
4f191ffc3586   slava1005/nginx:v1   "/docker-entrypoint.…"   13 seconds ago   Up 11 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   modest_banzai

slava@DESKTOP-QKJU13U:~/mynginx$ curl http://172.26.222.130
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m new DevOps Engineer!</h1>
</body>
</html>

Загрузим созданный образ в реестр Docker Hub:

slava@DESKTOP-QKJU13U:~/mynginx$ docker push slava1005/nginx:v1
The push refers to repository [docker.io/slava1005/nginx]
da7eb4620ffd: Pushed
72120687062c: Mounted from library/nginx
469fc702bc62: Mounted from library/nginx
74964efcae21: Mounted from library/nginx
ad4f5bc987ca: Mounted from library/nginx
ef050c9a03b5: Mounted from library/nginx
83c20bc61eb8: Mounted from library/nginx
1024e8977b69: Mounted from library/nginx
a0904247e36a: Mounted from library/nginx
v1: digest: sha256:c21243cc12cafed636e2a8fec39037dc5041f7d2841dc9c52d86b16014e92fb1 size: 2196
