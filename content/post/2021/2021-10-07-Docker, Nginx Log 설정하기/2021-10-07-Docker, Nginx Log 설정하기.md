---
title: Docker, Nginx Log 설정하기
description: 
date: 2021-10-07
image: 
categories:
    - Docker
tags:
    - Docker
    - Nginx
weight: 1
---

오늘 일하다 서비스가 중단되는 이슈를 봤었다. 이유는 공간부족이였는데, Log 데이터가 너무 많이 쌓여있어서 더 이상 저장하지 못해서 발생한 이슈였다. 그걸 보면서 문득 나는 Log를 관리하고 있었나...? 하는 생각이 들었다. Nginx에서 자동으로 쌓는 Log가 있다고 했는데, 어떻게 확인하는지도 찾아볼 겸 Log 설정하는 방법을 정리해보려고한다.

들어가기에 앞서서 나는 docker-compose를 이용해서 Nginx, FastAPI, Django를 서버에서 구동시키고 있다. 보통의 Nginx 설정과 다를 수 있음을 양해바란다.

Nginx에는 기본적으로 Log를 쌓는 기능이 있다. `/var/log/nginx/`에 access.log와 error.log가 쌓이게 된다. 이 설정 파일은 어디 있을까.

`/etc/nginx/nginx.conf`에 가면 nginx 설정 파일이 위치해있다. 나는 docker를 쓰니까 일단 `docker exec -it <container name> /bin/bash`로 접속해준다. 그 후 vi로 파일을 열어보면

```xml
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

이렇게 되있는데 파란색으로 표시되어 있는 부분이 log를 쌓는 공간이다. 그래서 가봤더니 파일이 열리지 않았다;;;;; 그래서 한참을 삽질하고 있었는데, 주황색 부분의 include에서 설정을 가져온다고 되있었다. 그게 뭔가...하고 가보면 conf.d/nginx.conf라고 있는데, 이건 내가 미리 작성한 후 docker-compose.yml에서 복사한 내용이다. 열어보면

```xml
server {
    listen 80;
    server_name localhost;
    charset utf-8;
    client_max_body_size 128M;

    location / {
        proxy_pass http://fastapi:8000/;
    }

    location /static/ {
        alias /static/;
    }
}

server {
    listen 80;
    server_name admin.localhost;
    charset utf-8;
    client_max_body_size 128M;

    location / {
        proxy_pass http://django:8001/;
    }

    location /static/ {
        alias /static/;
    }
}
```

nginx에서 도메인으로 서버를 분리하게 만들어 놓은 파일이다. 이제 여기에 각 서버별로 log를 쌓아주도록 하자. 아래와 같은 코드를 작성해주면된다. 위치는 자유지만 기본적으로 세팅되어 있는 곳에 같이 두기로 했다.
 나는 서버가 2개인 만큼 디렉토리를 분리해서 놓을 수도 있다. 그런데 docker 세팅할때 `mkdir` 하는 방법을 몰라서 일단 패스...ㅎ 이름으로 분리하도록 하자.

```xml
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;
```

아래와 같이 적용해주면, log를 쌓아주기 시작한다.

```xml
server {
    listen 80;
    server_name localhost;
    charset utf-8;
    client_max_body_size 128M;

    location / {
        proxy_pass http://fastapi:8000/;
    }

    location /static/ {
        alias /static/;
    }

    access_log /var/log/nginx/fastapi_access.log;
    error_log /var/log/nginx/fastapi_error.log;
}

server {
    listen 80;
    server_name admin.localhost;
    charset utf-8;
    client_max_body_size 128M;

    location / {
        proxy_pass http://django:8001/;
    }

    location /static/ {
        alias /static/;
    }

    access_log /var/log/nginx/django_access.log;
    error_log /var/log/nginx/django_error.log;
}
```

그러고보니 자동으로 백업이나 삭제하는 건 어떻게 하지...? 다음에 알아보자ㅎ