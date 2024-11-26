---
title: AWS EC2에 Docker-compose 설치하기
description: 
date: 2021-09-08
image: 
categories:
- AWS
tags:
- Docker Compose
weight: 1
---

## 환경

- Amazon Linux

## yum upgrade

```bash
sudo yum -y upgrade
```

## git install

```bash
sudo yum install git 
```

## docker & docker-compose install

```bash
# docker
sudo yum -y install docker
docker -v

# docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# docker run
sudo service docker start
sudo usermod -aG docker $USER
```

## docker-compose permission denied solution

```bash
ls -al /var/run/docker.sock
-> srw-rw----. 1 root docker 0 11월 23 17:15 /var/run/docker.sock

sudo chmod 666 /var/run/docker.sock

ls -al /var/run/docker.sock
srw-rw-rw-. 1 root docker 0 11월 23 17:15 /var/run/docker.sock
```

- 참고사이트
    > [Git](https://git-scm.com/download/linux)
    > [[Docker] Docker Compose 환경 구축, AWS EC2 도커 구축 실습](https://ozofweird.tistory.com/entry/Docker-Docker-%EC%8B%A4%EC%8A%B5-2)
    > [Install Docker Compose](https://docs.docker.com/compose/install/)
    > [PermissionError: [Errno 13] Permission denied](https://deeds-not-words.tistory.com/entry/PermissionError-Errno-13-Permission-denied)
