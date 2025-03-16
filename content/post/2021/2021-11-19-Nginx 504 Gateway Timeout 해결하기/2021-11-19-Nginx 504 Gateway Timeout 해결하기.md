---
title: Nginx 504 Gateway Timeout 해결하기
description: 
date: 2021-11-19
image: 
categories:
tags:
   - Nginx
weight: 1
---

새로운 프로젝트에서 Nginx - FastAPI를 이용해서 크롤링 관련된 기능을 만들고 있다.

Local에서는 실행시간이 조금 길어도 문제없어서 Timeout를 고려를 못하고 있다가, 서버 배포 후 Timeout이 발생하는걸 알게됬다. Nginx에서는 60초가 기본으로 세팅되있다. 이정도 일이면 사실 백그라운드로 돌려버려야하는데...ㅎ

쨋든 nginx.conf에서 timeoute을 늘려주려고 한다.

```bash
location / {
   proxy_connect_timeout 120;
   proxy_send_timeout 120; 
   proxy_read_timeout 120; 
   send_timeout 120; 
}
```

위와 같이 설정해주고, nginx를 재시작해주면 적용된다!

이 문제를 해결하려고 보면서, Nginx 같은 웹서버가 왜 필요한가를 다시 생각해보게 되었다. 사실 웹서버 없이 uvicorn-FastAPI로도 잘 돌아가긴 하니까...하지만 위와 같은 timeout 설정도 그렇고, 수많은 request를 처리해주는 능력이 훨씬 뛰어난 웹서버를 앞에 두는게 합리적인 선택이라고 한다. 웹서버와 관련된 내용은 나중에 한번 정리해야겠다.

> **참고**
> [https://install-django.tistory.com/22](https://install-django.tistory.com/22)
