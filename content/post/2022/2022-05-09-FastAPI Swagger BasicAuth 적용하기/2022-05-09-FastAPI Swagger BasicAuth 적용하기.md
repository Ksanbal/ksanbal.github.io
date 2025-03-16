---
title: FastAPI Swagger BasicAuth 적용하기
description: 
date: 2022-05-09
image: 
categories:
tags:
    - FastAPI
weight: 1
---

**FastAPI**는 기본적으로 `/docs /redoc /openapi.json`을  통해서 Swagger를 제공해준다. 자동으로 API 명세서를 작성해주기 때문에, 조금만 손을 보면 Front 개발자와 편하게 일을 할 수 있다.
하지만, 문제는 이 API 명세서는 밖으로 공개되면 안되는 매우 중요한 문서이다. 그동안 놓치고 있다가 nestJS 강의에서 Swagger를 붙이는 과정에서 배우면서 문득 큰일났다 싶어서, 빠르게 찾아서 적용해봤다.

 공식문서에 나와있는 방법은 아니고, 구글링과 공식문서의 인증 부분을 합쳐서 간단하게 적용해봤다.

> 참고1.
> 
> 
> [https://nilsdebruin.medium.com/fastapi-how-to-add-basic-and-cookie-authentication-a45c85ef47d3](https://nilsdebruin.medium.com/fastapi-how-to-add-basic-and-cookie-authentication-a45c85ef47d3)
> 
> [HTTP Basic Auth - FastAPI](https://fastapi.tiangolo.com/advanced/security/http-basic-auth/)
> 

## 아이디어

- 기본적인 아이디어는 이렇다. 자동으로 생성되는 docs, openbapi.json, redoc을 비활성화 시킨다.
- docs로 접근하면 아이디 & 비밀번호를 체크한다.
- 아이디 & 비밀번호가 일치하면 Swagger를 반환한다.
- 끝

## 구현

### 기본 swagger 비활성화

- 기본 swagger를 비활성화하는 이유는 활성화되어 있으면 내가 만든 router를 거치지 않기 때문이다. 기본적으로 auth 기능이 있는 것 같긴하지만 공식문서에도 정리가 되어있지 않아서 찾을 수 없었다;;;

```python
app = FastAPI(docs_url=None, redoc_url=None, openapi_url=None)
```

### 아이디 & 비밀번호 체크 함수 작성

- dependencies로 사용할 비밀번호 체크 함수를 작성한다. 여기서 인증 방식은 db에 저장된 방식을 쓸 수도 있고, 자유롭게 선택할 수 있지만 나는 .env에 값을 저장하고 사용하는 방식으로 간단하게 만들어봤다.

```bash
# .env
SWAGGER_USER=admin
SWAGGER_PASSWORD=superSecret
```

```python
from fastapi.security import HTTPBasic, HTTPBasicCredentials
import os

def checkSecure(credentials: HTTPBasicCredentials = Depends(HTTPBasic())):
    if credentials.username == os.getenv('SWAGGER_USER') and credentials.password == os.getenv('SWAGGER_PASSWORD'):
        return Response(status_code=200)
    raise HTTPException(status_code=401,detail='invalid account')
```

### Swagger router 적용

- 위에 작성한 함수를 dependencies로 적용하고 swagger를 반환하자.
여기서 포인트는 openapi.json을 이용해서 swagger ui를 만들어주는 함수를 사용하기 때문에 `/docs`뿐만 아니라 `/openapi.json`도 꼭 활성화시켜줘야한다.

```python
from fastapi.openapi.docs import get_swagger_ui_html
from fastapi.openapi.utils import get_openapi

@app.get('/openapi.json', dependencies=[Depends(checkSecure)])
def getOpenAPI():
    return JSONResponse(get_openapi(title='TestAPI', version=1, routes=app.routes))

@app.get('/docs', dependencies=[Depends(checkSecure)])
def getDocs():
    return get_swagger_ui_html(openapi_url='/openapi.json', title='docs')
```

### 그러면 뺌!

![스크린샷 2022-05-10 오전 9.15.39.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/90039b65-d9f8-43d9-86c8-90edcde8c671/스크린샷_2022-05-10_오전_9.15.39.png)

- Full code