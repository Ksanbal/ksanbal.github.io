---
title: Django, Gunicorn, Nginx, Docker 설정
description: 
date: 2021-09-08
image: 
categories:
- Docker
tags:
- Django
- Gunicorn
- Nginx
- Docker
weight: 1
---

## Django, Gunicorn

1. gunicorn 설치 및 테스트
    - `pip install gunicorn`
    - `gunicorn —bind 0.0.0.0:8000 [project_name].wsgi:application`
2. `pip freeze > requirements.txt`
3. dockerfile
    
    ```python
    # python version
    FROM python:3.9.6
    
    # 작업 디렉토리
    WORKDIR /usr/src/app
    
    # requirements 이동 및 설치
    COPY requirements.txt ./
    RUN pip install --upgrade pip
    RUN pip install -r requirements.txt
    
    # Project 복사
    COPY . .
    
    # *포트 설정 & gunicorn 실행은 docker-compose에서 작성*
    # 포트 설정
    # EXPOSE 8000
    
    # gunicorn 실행
    # CMD ["gunicorn", "--bind", "0.0.0.0:8000", "Django_Docker.wsgi:application"]
    ```
    
4. `docker build -t [account]/[image name]:[version] .`
    - account : 사용자명, option 인 것 같다
    - image name : 이미지명
    - version : latest로 진행
5. `docker run -it -d -p 8000:8000 —name [build 명] [account]/[image name]`

## Nginx
- nginx.conf
    ```yaml
    # [Django Project Name]/.config/nginx/nginx.conf
    # docker-compose에서 작성한 서비스 이름이 web
    upstream web {
        ip_hash;
        server web:8000;
    }
    
    server {
        listen 80;
        server_name localhost;
        charset utf-8;
        client_max_body_size 128M;
    
        location / {
            proxy_pass http://web/;
        }
    
        location /static/ {
            alias /static/;
        }
    }
    ```
- static 파일 처리하기
    
    ```python
    # settings.py
    ...
    BASE_DIR = Path(__file__).resolve().parent.parent
    ...
    STATIC_URL = '/static/'
    STATIC_DIR = os.path.join(BASE_DIR, 'static')
    STATICFILES_DIRS = [
        STATIC_DIR,
    ]
    STATIC_ROOT = os.path.join(BASE_DIR, '.static')
    ...
    ```
    

## docker-compose

1. docker-compose.yml
    
    ```yaml
    version: "3"
    services:
      nginx:
        image: nginx:latest
        container_name: nginx
        ports:
        - "80:80/tcp"
        volumes:
        - ./.config/nginx:/etc/nginx/conf.d
        - ./.static:/static
        depends_on:
          - web
    
      web:
        build:
          context: .
          dockerfile: dockerfile
        container_name: web
        command: gunicorn Django_Docker.wsgi:application --bind 0.0.0.0:8000
        volumes:
        - ./static:/user/src/app/staticfiles
        expose:
          - "8000"
    ```
    
2. docker-compose up
    - `docker-compose up --build` : image까지 같이 생성

## 환경변수 설정 feat.DB info

Django의 DB 정보를 환경변수로 등록하여 사용하기 위해 필요

- .env
    - 최상위 프로젝트 폴더에 저장
    - 띄어쓰기가 있으면 안됨
    
    ```bash
    DB_ENV_MARIADB_USER="계정 아이디"
    DB_ENV_MARIADB_PASSWORD="계정 비밀번호"
    DB_ENV_MARIADB_HOST="HOST 주소"
    ```
    
- docker-compose.yml
  1. 자동으로 인식해서 가져오는 방법

  ```yaml
  api:
      build:
        context: .
        dockerfile: dockerfile-dev
      container_name: dayou_api
      command: gunicorn Django_Docker.wsgi:application --bind 0.0.0.0:8000
      environment:
        DB_ENV_MARIADB_USER: "${DB_ENV_MARIADB_USER}"
        DB_ENV_MARIADB_PASSWORD: "${DB_ENV_MARIADB_PASSWORD}"
        DB_ENV_MARIADB_HOST: "${DB_ENV_MARIADB_HOST}"
      volumes:
      - ./static:/user/src/app/staticfiles
      expose:
        - "8000"
  ```
    
 1. env 파일을 명시해주는 방법

 ```yaml
 api:
     build:
       context: .
       dockerfile: dockerfile
     container_name: web
     command: gunicorn Django_Docker.wsgi:application --bind 0.0.0.0:8000
     env_file:
	    - .env
     volumes:
 	    - ./static:/user/src/app/staticfiles
     expose:
       - "8000"
 ```
    
- Django settings.py
    - os.getenv('환경변수')로 변수값 가져오기
    ```python
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'mysql',
            'USER': os.getenv('DB_ENV_MARIADB_USER'),
            'PASSWORD': os.getenv('DB_ENV_MARIADB_PASSWORD'),
            'HOST': os.getenv('DB_ENV_MARIADB_HOST'),
            'PORT': '3306',
        }
    }
    ```