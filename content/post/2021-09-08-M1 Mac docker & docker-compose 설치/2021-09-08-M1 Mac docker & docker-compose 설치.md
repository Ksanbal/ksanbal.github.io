---
title: M1 Mac docker & docker-compose 설치
description: 
date: 2021-09-08
image: 
categories:
- Docker
tags:
weight: 1
---

# Install

---

## Brew

- `brew install cask docker`
    - Docker Desktop on Mac을 설치
    - docker-compose, docker-machine을 같이 설치
    - 포트포워딩 불필요
- `brew install docker`
    - 가상머신 위에 도커를 띄우는 작업을 해야한다.
    - docker-compose, docker-machine을 추가로 설치
    - 포트포워딩 필요

## Apple Silicon
[Docker Desktop for Mac and Windows | Docker](https://www.docker.com/products/docker-desktop)

# 기본 명령어

- 참고사이트
    > [Docker Compose 커맨드 사용법](https://www.daleseo.com/docker-compose/)