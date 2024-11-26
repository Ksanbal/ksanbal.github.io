---
title: FastAPI 시작하기 (1) feat. Django
description: 
date: 2021-09-08
image: 
categories:
tags:
weight: 1
---

 새로운 프로젝트에 들어갈 준비를 하면서, 백엔드 프레임워크에 대한 고민을 많이 했었다. 그동안 프로젝트들은 모두 Django로 진행했었는데, 지금 생각해보면 항상 그 상황에 최선의 선택은 아니였을지도 모르겠다. 이번 프로젝트에 다른 프레임워크도 후보군에 있었지만, 어쩌면 최종적으로 결정될...? 프레임워크인 FastAPI를 소개해보려고한다.

 개인적으로 새로운 프레임워크를 배울때 다양한 블로그와 유튜브를 참고하면서 배우는데, 오히려 Django를 배울때보다 FastAPI를 배우는게 편했던 것 같다. stack overflow는 Django가 잘되있는데, 처음 배울때 공식문서나 유튜브가 잘되있는 느낌? 

## 그래서 FastAPI가 뭔가요?

 [공식페이지](https://fastapi.tiangolo.com/ko/)에 의하면 현대적이고, 빠르며 파이썬 표준 타입 힌트에 기초한 Python 3.6+의 API라고 한다. 다른 부분들보다 **빠르다!**라는 부분이 인상적이였다. NodeJS나 Go와 대등할 수준이라고 한다. (다른 블로그에서 속도를 비교해주신게 있는데, 무조건 그 수준인건 아닌 것 같지만 확실히 실행속도나 처리에서 체감상 Django보다는 빠른 것 같다.) 그 외에 빠른 코드작성, 적은 버그, 직관적, 쉬움, 짧음과 같은 특징들이 있지만 내가 느끼는 또 다른 큰 강점은 OpenAPI이랑 JSON스키마를 이용한 문서 자동화가 아닐까 싶다.

- **FastAPI의 장점**
    - 빠르다
    - 코드작성도 빠르다
    - 버그가 적다
    - 편집기 지원과 자동완성이 직관적이다
    - 배우기 쉽고, 문서가 잘 되있다
    - API 문서가 자동으로 생긴다.

FastAPI의 빠른 속도는 [Starlette](https://www.starlette.io/) 때문이라고 하는데 Django나 Flask의 WSGI 서버보다 빠른 ASGI 서버이기 때문이라고 한다. 비동기처리가 가능하고 등등 뭐가 많았는데, 사실 아직까진 잘 모르겠다...ㅎ 처리가 큰 프로세스에 한해서 비동기처리가 동기처리보다 좋은 것으로 알고 있는데, 대신 클라이언트에서 Callback 처리를 해줘야한다고 한다.

 깊게 들어가면 잘 모르는거 티나니까 얼른 코드로 넘어가야겠다...ㅎ

## 코드로 얘기합시다...ㅎ

`python -m venv venv`로 가상환경을 만들고 아래 코드로 설치해준다.

```bash
pip install fastapi
pip install uvicorn
```


- 근데 쓰다보니 **requirements.txt**를 쓰는게 더 편한 것 같다. 
  - 전에는 pip로 다 설치하고서 나중에 freeze 시켜서 requirements.txt를 만들었었는데, 먼저 만드는게 오히려 편한 것 같다...
    ```
    # requirements.txt
    
    fastapi
    uvicorn
    ```
    
    ```bash
    pip install -r requirements.txt
    ```
    

```python
# main.py
from fastapi import FastAPI

from typing import Optional

app = FastAPI()

@app.get("/") # url route 설정
def read_root():
		return {"Hello": "World!"}

@app.get("/item/{item_id}") # url로 변수 받기
def read_root(item_id: int, q: Optional[str]=None):
		# 여기서 q는 url 쿼리로 받게된다.
		return {"item_id": item_id, "q": q}
```

- 다양한 실행방법...?
    1. Terminal에서 바로 시작하기
        
        ```bash
        uvicorn main:app --host 0.0.0.0 --port 80 --reload
        
        INFO:     Started server process [19095]
        INFO:     Waiting for application startup.
        INFO:     Application startup complete.
        INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
        ```
        
    2. vscode에서 launch.json을 수정하기
        - vscode의 **Run and Debug**에 가면 `create a launch.json file.`을 클릭하고 FastAPI를 선택해주면 .vscode/launch.json이 생성된다. 이번에 처음 작성해 봤는데 명령어의 띄어쓰기를 기준으로 다 나눠줘야한다.~~(왜 불편한데 편하지?)~~
        
        ```json
        {
            // Use IntelliSense to learn about possible attributes.
            // Hover to view descriptions of existing attributes.
            // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
            "version": "0.2.0",
            "configurations": [
                {
                    "name": "Python: FastAPI",
                    "type": "python",
                    "request": "launch",
                    "module": "uvicorn",
                    "args": [
                        "main:app",
        								"--host",
        								"0.0.0.0",
        								"--port",
        								"80"
        								"--reload"
                    ],
                    "jinja": true
                }
            ]
        }
        ```
        
    3. `python main.py`로 실행하기
        - 다음과 같이 코드를 수정하고 `python main.py` 을 실행하면된다. [참고했던 블로그](https://dingrr.com/blog/post/python-fastapi-%EB%A1%9C-%EB%B0%B1%EC%97%94%EB%93%9C-%EB%A7%8C%EB%93%A4%EA%B8%B0-2%ED%99%94-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EA%B5%AC%EC%A1%B0)에서는 함수로 한번 더 묶어줬었는데, 나는 코드 수정하면서 reload 될때 main.py에 에러표시 뜨면서 옮겨지는게 거슬려서 쓰고있진않다. 나중에 배포단계에서는 명확해서 좋을 듯...
        
        ```python
        # main.py
        import uvicorn
        from fastapi import FastAPI
        
        from typing import Optional
        
        app = FastAPI()
        
        @app.get("/")
        def read_root():
        		return {"Hello": "World!"}
        
        @app.get("/item/{item_id}")
        def read_root(item_id: int, q: Optional[str]=None):
        		return {"item_id": item_id, "q": q}
        
        if __name__ == "__main__":
        		uvicorn.run("main:app", host="0.0.0.0", port=80, reload=True))
        ```
        

이제 [localhost:8000](http://localhost:8000/)로 접속해보면 

![스크린샷 2021-09-08 오후 8.07.47.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3e67f14b-e5be-4737-a8bd-b0b50fb9d08b/스크린샷_2021-09-08_오후_8.07.47.png)

이렇게 return을 받아 볼 수 있다. Flask는 개인적으로 뭔가 배우기 어려워서 배우질 못했었는데, 시작은 이런 느낌이였던걸로 기억한다. 하지만 FastAPI의 매력은 여기가 아니라

[locahost:8000/docs](http://localhost:8000/docs)랑 [locahost:8000/redoc](http://locahost:8000/redoc) 여기라고 생각한다...

앞서 이야기했던 OpenAPI(SwaggerUI)와 JSON스키마가 바로 이것이다.

![스크린샷 2021-09-08 오후 8.10.20.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f5d86cf6-1b95-4e21-a3eb-ef6a6c1cc1b7/스크린샷_2021-09-08_오후_8.10.20.png)

![스크린샷 2021-09-08 오후 8.10.37.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a93ce90b-7e62-4462-a354-5f9db4058665/스크린샷_2021-09-08_오후_8.10.37.png)

백엔드 작업하고서 가장 스트레스 받으면서 만들었는데 방치되기 가장 쉬운 문서가 이런 API 인터페이스 정의서라고 생각한다. 만들때 노가다하는데 프로젝트 중간중간 수정되는걸 다 적용시키는게 진짜 만만치않다... 이 자동문서화도 뭔가 커스터마이징과 더 자세하게 보여지게하는 것들이 가능할 것 같은데, 필요할때 천천히 찾아볼 생각이다.

OpenAPI는 Try it out으로 request를 테스트해볼 수 있는데, 개발단계에서 빠르게 타닥타닥하면서 테스트하기엔 진짜 편하다

![스크린샷 2021-09-08 오후 8.15.07.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f3b46656-d9ca-41e2-9592-def82cce3230/스크린샷_2021-09-08_오후_8.15.07.png)

## 마무리

 지인짜 간단하게 맛만 봤는데, 이번에 실제 프로젝트에 적용할 예정이라 앞으로 조금 더 실질적인 부분들을 다룰 예정이다. Django로 이미 거의 다 만들어 놓은 프로젝트를 FastAPI로 옮기는 작업을 할 예정이라 둘의 차이를 비교하면서 보는 재미가 쏠쏠하지 않을까...싶다!
