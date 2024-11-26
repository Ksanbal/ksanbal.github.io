---
title: Nginx & Docker Local에서 멀티 도메인 지원하기(FastAPI에서 Django Admin 쓰기)
description: 
date: 2021-09-16
image: 
categories:
- Docker
tags:
weight: 1
---

프로젝트를 Django에서 FastAPI로 바꾸면서 생기는 문제가 하나 있었는데, 바로 Admin 페이지였다.
FastAPI에서도 Django Admin에 영감을 받아서 FastAPI admin이 있긴한데, 아직 적용과 사용법이 잘 정리되어 있지 않다는 느낌을 받았다.  게다가 Django Admin은 진짜 너무 편하기도하고...이미 이래저래 설정해놨는데 날리는게 좀 아까웠다.

 그래서 생각한게 어차피 docker를 쓰니까 Django contianer를 하나 더 올려서 따로 구동시키는 것이다. 그런데 문제는 그걸 어케함...?

## 정답은 Nginx에 있었다!

 Nginx는 웹서버 소프트웨어이다. Django만 쓸때는 웹서버로만 사용했었는데 로드밸런싱, 리버스 프록시 등의 기능을 수행할 수 있다고 한다. 이 Nginx 설정 삽질로 얻은 방법을 공유하려고한다.

## 그래서 방법이?

1. 일단 FastAPI와 Django를 포트만 다르게 구동시킨다.
    
    > FastAPI : localhost:8000
    Django : localhost:8001
    > 
2. Nginx에서 8000번대 포트를 80번으로 열어준다.
3. 서브 도메인 방식을 사용해서 Nginx를 설정한다.

그러면 `docker-compose.yml`와 `nginx.conf`를 한번 보자

### docker-compose.yml

```yaml
version: "3"
services:
  nginx:
    image: nginx:latest
    container_name: nginx
    ports:
    - "80:80"
    volumes:
    - ./.config/nginx:/etc/nginx/conf.d # nginx 설정 파일
    - ./.static:/static # Django Admin의 css을 위해 static 설정
    depends_on:
      - fastapi
      - django

  fastapi:
    build:
      context: .
      dockerfile: dockerfile-fastapi
    container_name: fastapi
    command: uvicorn main:app --host 0.0.0.0 --port 8000 # 8000번 포트로 서버 실행
    env_file:
      - .env
    expose:
      - "8000"
    depends_on:
      - db

  django:
    build:
      context: .
      dockerfile: dockerfile-django
    container_name: django
    command: gunicorn main.wsgi:application -b 0.0.0.0:8001 # 8001번 포트로 서버 실행
    env_file:
      - .env
    expose:
      - "8001"
    depends_on:
      - db
    deploy:
			# Admin 페이지용으로만 쓰일 것을 고려해서 cpu & memory limit 설정
      resources:
        limits:
          cpus: "0.25"
          memory: 512M

  db:
    image: postgres:latest
    container_name: postgresql
    ports:
      - "5432:5432"
    expose:
      - "5432"
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
      POSTGRES_DB: postgre
```

### nginx.conf

```jsx
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

 내가 선택한 방법은 서브도메인을 이용하는 방법이였다. 같은 `server{}`내에서 `location`만 다르게 지정해서 하는 방법도 있었는데, 이렇게 하면 실행은 된다. 하지만 url에 문제가 생기면서 다른 페이지들을 불러오지 못해서 패스...
 남은 방법은 서브도메인이였는데, 문제는 아직 개발중이라 도메인이 없다는 점;;; 그래서 못하나했더니 `localhost`에도 서브도메인이 적용된다! 그래서 Django는 `admin.localhost`로 지정해줬다. `localhost`가 도메인처럼 쓰이는 거라 가능했나보다 ~~(admin.0.0.0.0이나 admin.127.0.0.1 같은걸 시도하고 있었다니...)~~

 그래서 [http://localhost/docs](http://localhost/docs) 로 접속하면 FastAPI의 Document가
[http://admin.localhost/](http://admin.localhost/) 로 접속하면 Django의 Admin 페이지가 보여지게 된다.