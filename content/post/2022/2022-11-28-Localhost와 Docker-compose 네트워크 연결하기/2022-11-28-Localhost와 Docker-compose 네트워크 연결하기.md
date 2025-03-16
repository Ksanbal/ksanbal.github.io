---
title: Localhost와 Docker-compose 네트워크 연결하기
description: 
date: 2022-11-28
image: 
categories:
    - Docker
tags:
    - Docker
    - Docker Compose
weight: 1
---

docker-compose 로 서버 관리를 하면서 불편한 점이 하나 있었다. 바로 컨테이너 업데이트가 있을때 전체 docker-compose up --build 명령어를 이용해야했다는 점이다. 일단 속도도 느리고, db 같은 컨테이너들은 volumnes로 묶여있는 폴더가 같이 있으면 오류를 발생시키거나 그랬다. 하지만 nginx, database, redis와 같이 업데이트가 자주 일어나지 않는 컨테이너들을 관리하고 올리는데 docker-compose만큼 편한 도구가 또 없다...
그래서 생각한게  **web server만 local에서 실행시키고, 나머지를 docker로 돌리면 안되나?** 였다. 하지만 docker는 내부 네트워크 default로 서로 연결되어 있기 때문에 docker 내에서의 localhost는 우리가 생각하는 local이 아니다. 그렇다면 어떻게 해야할까?

## host.docker.internal 이용하기
- docker 내부에서 host.docker.internal로 아래와 같이 내 PC localhost로 연결할 수 있다.
```nginx
server {
    listen 80;
    server_name localhost;
    charset utf-8;
    client_max_body_size 128M;

    location / {
        proxy_pass http://host.docker.internal:3000;
    }

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
}
```

- 추가로 linux 환경에서 docker는 docker-compose에 따로 host.docker.internal를 사용한다는 내용을 추가해줘야한다.
	```yaml
	nginx:
		...
		extra_hosts:
	      - "host.docker.internal:host-gateway"
	```

끝!

> 참고
> [https://shanepark.tistory.com/209](https://shanepark.tistory.com/209)