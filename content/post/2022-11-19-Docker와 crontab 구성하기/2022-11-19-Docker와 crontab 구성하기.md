---
title: Docker와 crontab 구성하기
description: 
date: 2022-11-19
image: 
categories:
    - Docker
tags:
    - Docker
    - FastAPI
weight: 1
---

프로젝트를 진행하다보면 꼭 crontab을 이용해 스케쥴링을 하게되는 일이 있는 것 같다. 사실 python의 스케쥴링에는 celery도 있고, Django는 django-crontab이 있어서 그걸 이용하면 된다. 

하지만 난 FastAPI-Docker 조합이여서, 다른 방법을 찾아야했다.

## 1. fastapi-utils 사용하기

[fastapi-utils](https://fastapi-utils.davidmontague.xyz/) 는 fastapi에서 사용가능한 여러 부가적인 기능들을 제공해주는 패키지이다. 그 안에 Repeated Tasks라는 반복작업을 수행해주는 기능이 있다. 아래가 샘플코드인데, 함수에 @repeat_every() 라는 데코레이터만 설정해주면 된다. async sync와 관련없이 다 사용이 가능하다고 하는데, 뭔가 나의 경우에는 제대로 작동하지 않아서 포기했다.

```python
from fastapi import FastAPI
from sqlalchemy.orm import Session

from fastapi_utils.session import FastAPISessionMaker
from fastapi_utils.tasks import repeat_every

database_uri = f"sqlite:///./test.db?check_same_thread=False"
sessionmaker = FastAPISessionMaker(database_uri)

app = FastAPI()

def remove_expired_tokens(db: Session) -> None:
    """ Pretend this function deletes expired tokens from the database """

@app.on_event("startup") # app이 실행될때 함수를 실행시키는 데코레이터
@repeat_every(seconds=60 * 60)  # 1 hour
def remove_expired_tokens_task() -> None:
    with sessionmaker.context_session() as db:
        remove_expired_tokens(db=db)
```

## 2. Crontab 이용하기

Linux에는 crontab이 기본적으로 설치되어있다. Docker Container 내부나 따로 Cron만 하는 Container를 설정하는 방법도 있다고 했지만, 난 그렇게 하진 않고, OS에서 cron을 걸어주는 방식으로 진행했다.

 아래 명령어는 Docker Container 안으로 접속할때 쓰는 명령어이다. 참고했던 블로그에서 아래의 방식을 응용해서 Crontab을 걸어주는 방법이다.

```python
docker exec -it [Container Name] /bin/bash
```

그러고 Container안에 스케쥴링때마다 실행시킬 code를 작성한다.

```python
# cron.py

import requests
from datetime import datetime

response = requests.get("http://127.0.0.1:8000/cron-test")
print(f'{datetime.now()} : crawl cron status({response.status_code})')
```

그 다음 Crontab에서 다음과 같이 명령어를 작성해주는 것이다.

```bash
* * * * * docker exec [Container Name] python /usr/src/app/cron.py >> /home/ec2-user/cron.log 2>&1
```

-i와 -t 명령어를 빼준 이유는 interactive와 terminal이 필요없는 명령어이기 때문에 빼주었다.

끝!

> **참고**
[https://jdm.kr/blog/2](https://jdm.kr/blog/2)
[https://yujuwon.tistory.com/entry/docker에서-crontab-동작](https://yujuwon.tistory.com/entry/docker%EC%97%90%EC%84%9C-crontab-%EB%8F%99%EC%9E%91)
>