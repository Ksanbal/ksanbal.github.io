---
title: AWS - IoT Core 설정법
description: 
date: 2021-09-08
image: 
categories:
- AWS
tags:
- AWS IoT Core
weight: 1
---

# Lambda로 IoT Core topic subscribe

1. Trigger 추가
   1. Trigger : AWS IoT Core
   2. 사용자 지정 IoT 규칙
     1. 규칙 쿼리 설명문 : `SELECT * FROM 'DT/~~~'`
2. Layer(계층) 생성
   - Python 패키지가 설정되어 있지않기 때문에, Layer에 패키지를 업로드해야한다.
   1. 계층 → 계층성
     1. 업로드
         [request_bs4_lxml-12c6bd1c-696b-47d8-8e44-b5166b69ec5f.zip](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/db001065-afb2-462b-9755-25a56b846349/request_bs4_lxml-12c6bd1c-696b-47d8-8e44-b5166b69ec5f.zip)
         
     2. 런타임 설정
         1. 첨부된 파일은 Python 3.6에서 실행가능
   2. Layer 생성없이 하는 방법
     3. boto가 request를 내장하고 있기 때문에 import 가능
     4. `from botocore.vendored import requests`
     5. python 3.8에서는 실행이 안됬는데, 3.6에서는 가능함
         - 2021-12-01 삭제예정...?
             - urlib3를 쓰고싶어서 requests를 가지고 있었는데, 다양한 버전에 적용이 안된다고, 제거하고 따로 설치해야하게 바뀐다고 한다.
             [Removing the vendored version of requests from Botocore | Amazon Web Services](https://aws.amazon.com/ko/blogs/developer/removing-the-vendored-version-of-requests-from-botocore/)

3. event 값을 http로 전송
    
    ```python
    import json
    import requests
    
    def lambda_handler(event, context):
        requests.post(
            'https://www.ksanbal.com/api/lambda/',
            data=json.dumps(event),
            headers={
                "Content-Type": "application/json",
            }
        )
    ```
    

# Django(python)에서 AWS IoT Core로 topic publish 하기

- python에서 boto3 package를 이용하면 AWS의 대부분 기능들을 사용할 수 있다.
- boto3를 사용하기 위해선, Access key와 Region 설정이 필요하다
  - `pip install boto3`
  - Access key 설정
      ```json
      // linux & Mac : ~/.aws/credentials
      // Window : C:\Users\USER_NAME\.aws\credentials)
      [default]
      aws_access_key_id = YOUR_KEY
      aws_secret_access_key = YOUR_SECRET
      ```
      
  - Region 설정
      
      ```json
      // linux & Mac : ~/.aws/config
      // Window : C:\Users\USER_NAME\.aws\config)
      [default]
      region=ap-norteast-2
      ```
      
  - Docker 를 사용하는 사람을 위한 설정법
      - 프로젝트에 Docker-compose를 이용하고 있는데, 이 설정파일들은 .yml 파일에서 env_file로 지정해줘도 적용이 안되더라. 그래서 volumes로 복사시켜서 적용시켰다.
      
      ```docker
      ...
      volumes:
      - ./.aws:/root/.aws
      ```

- 명령어
    ```python
    import boto3
    import json
    
    client = boto3.client('iot-data')
    
    response = client.publish(
        topic='DT/ABCD',
        qos=1,
        payload=json.dumps({
            "message": "Hello from EC2 Django"
        })
    )
    ```
    
    - error : SSL validation failed for
        - Proxy 서버를 통한 통신이 아니여서 발생하는 오류 같음
        - `boto3.client('iot-data', verify=False)`로 해결할 수 있었으나, 보안적인 이슈가 있지는 않을까...고민되는 부분이다.
- 참고
    > [Python용 AWS SDK (Boto3) 정식 출시 | Amazon Web Services](https://aws.amazon.com/ko/blogs/korea/now-available-aws-sdk-for-python-3-boto3/)
    > [IoT - Boto3 Docs 1.18.33 documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iot.html)