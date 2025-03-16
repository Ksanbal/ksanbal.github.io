---
title: 서버 실행시 Docker-compose 자동 실행하기
description: 
date: 2021-11-18
image: 
categories:
- Docker
tags:
- Docker Compose
weight: 1
---

서버가 껐다 켜질때마다 docker를 켜고 docker-compose up 을 해주는 건 사실 까먹기 쉬우니까 자동으로 실행하도록 system에 등록하기로 하자.

- 실행문서 작성하기

```bash
# 파일을 새로 작성하는 것이기 때문에, 권한이 걸리는 경우가 있어서 sudo를 붙여줬다.
sudo vim /etc/systemd/system/docker-compose.service
```

```bash
[Unit]
Description=Docker Compose Application Service
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=[docker-compose up을 실행할 디렉토리 (작업경로...?)]
ExecStart=/usr/local/bin/docker-compose up -d # 시작시 실행시킬 명령어
ExecStop=/usr/local/bin/docker-compose stop # 종료시 실행시킬 명령어
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

파일을 다 작성했다면, 아래 명령어로 시스템에 등록해준다.

```bash
sudo systemctl enable docker-compose
```

끝!

> **참고**
> [[docker] 서버 실행 시 docker-compose 자동 실행](https://velog.io/@1yangsh/docker-%EC%84%9C%EB%B2%84-%EC%8B%A4%ED%96%89-%EC%8B%9C-docker-compose-%EC%9E%90%EB%8F%99-%EC%8B%A4%ED%96%89)
>