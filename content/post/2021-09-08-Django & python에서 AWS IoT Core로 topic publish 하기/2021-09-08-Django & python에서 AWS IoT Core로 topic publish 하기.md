---
title: Django & python에서 AWS IoT Core로 topic publish 하기
description: 
date: 2021-09-08
image: 
categories:
- AWS
tags:
- AWS IoT Core
- Django
weight: 1
---

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
    [Python용 AWS SDK (Boto3) 정식 출시 | Amazon Web Services](https://aws.amazon.com/ko/blogs/korea/now-available-aws-sdk-for-python-3-boto3/)
    [IoT - Boto3 Docs 1.18.33 documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iot.html)
