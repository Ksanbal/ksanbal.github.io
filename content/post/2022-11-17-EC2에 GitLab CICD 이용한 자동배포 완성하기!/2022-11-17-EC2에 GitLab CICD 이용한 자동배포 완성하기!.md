---
title: EC2에 GitLab CICD 이용한 자동배포 완성하기!
description: 
date: 2022-11-17
image: 
categories:
- AWS
tags:
    - AWS EC2
    - GitLab
    - CI/CD
weight: 1
---

전부터 CI/CD를 구성하고 싶었는데, Jenkins니 Kubernetes니 뭔가 어렵게 느껴져서 쉽게 적용하지 못하고 있다가, 우연히 GitLab CI/CD 적용법을 알게되서 적용해보게 되었다. Gitlab을 사용하는 회사에서는 자연스럽게 GitLab CI/CD를 선택한다니까, 믿고 쓸만한 것 같다. 완전 무료는 아닌 것 같지만, 무료로 제공해주는만큼만 써도 그게 어디야…

# 구조

내가 그동안 배포를 진행하던 방식은 이렇다.

### 기존 배포방식

1. GitLab으로 코드를 `push`한다.
2. ec2에서 `git pull`을 이용해 코드를 가져온다.
3. `docker-compose up —build -d` 로 docker를 build시킨다.
4. 끝

### GitLab CI/CD 적용

1. GitLab으로 코드를 `push`한다.
2. GitLab이 미리 작성한 `.gitlab-ci.yml`와 등록된 `Runner`가 `Pipeline`을 실행한다.
3. `gitlab-runner`가 설치된 ec2에서 정의된 명령어를 수행한다.
4. 끝

이렇게 보니까 그렇게 간단해진건 아닌 것 같지만, 나는 앞으로 push만 하면 되는거니까 개꿀…

# EC2에 gitlab-runner 설치하기

1. GitLab-CI는 설치형이기 때문에 EC2에 해당 프로그램을 설치해야한다.
    
    ```bash
    $ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
	$ sudo yum install gitlab-runner
    ```

2. yum으로 설치하고나면 gitlab-runner라는 이름으로 이미 서비스가 등록되어버린다. 적용된 서비스의 삭제를 먼저 진행하자.
	```shell
	# 모든 runner 삭제
	$ gitlab-runner unregister --all-runners
	# gitlab-runner 제거
	$ sudo gitlab-runner uninstall
	# 추가된 사용자 제거
	$ sudo userdel gitlab-runner
	$ sudo rm -rf /home/gitlab-runner/
	```
    
3. 명령어를 실행하게될 유저와 directory를 설정해줘야하는데, 나는 이미 코드가 ec2-user에 있었기 때문에 root로 접근시켜서 해결하는 방식으로 했다. gitlab-runner라는 이름으로 user를 생성해서 하는 것도 방법이다.
    
    ```bash
    $ sudo gitlab-runner install --user=root --working-directory=/home/ec2-user
    $ sudo gitlab-runner start
    ```
    
4. 이제 GitLab으로 runner를 등록해주자. Token 입력이 필요하니까 미리 GitLab의 Settings → CI/CD → Runners에 Specific runners를 열어두자.
    
    ```bash
    $ sudo gitlab-runner register
    
    # GitLab 서버 주소 입력
    Enter the GitLab instance URL (for example, https://gitlab.com/):
    https://gitlab.com/
    
    # GitLab CI Token 입력 (Specific runners에 친절하게 적혀있다)
    Enter the registration token:
    ...
    
    # Runner 설명
    Enter a description for the runner:
    gitlab-runner for my gitlab repository
    
    # .gitlab-ci.yml에 작성될 tag인데, 꼭 기억하고 있어야한다. 
    Enter tags for the runner (comma-separated):
    deploy
    
    # 이건 뭔지 모르겠다...일단 패스ㅎ
    Enter optional maintenance note for the runner:
    
    # 실행할 명령 형식을 선택하는건데, 그동안 배포를 shell로 진행했으니까 shell로 선택
    Please enter the executor: docker-ssh, ssh, virtualbox, docker, parallels, shell, docker+machine, docker-ssh+machine, kubernetes:
    shell
    ```
    
5. 간혹 등록한 후에 ERROR: Preparation failed: getwd: getwd: no such file or directory 에러가 발생하는 경우가 있다. 이걸 방지하기 위해서 restart를 진행해주자.
    [ERROR: Preparation failed: Getwd: getwd: no such file or directory](https://stackoverflow.com/questions/57328978/error-preparation-failed-getwd-getwd-no-such-file-or-directory)
	```shell
	$ gitlab-runner restart
	```

6. 미리 열어둔 Settings → CI/CD → Runners에 가보면 runner가 등록된 것을 확인할 수 있다.

# `.gitlab-ci.yml` 작성하기

repository 최상단에 들어가게될 .gitlab-ci.yml을 작성해주자! runner가 실행될때 실제로 돌아게될 코드들을 작성하면 된다. 공식문서를 참고한건 아니고…여러 블로그를 조합한 결과이기 떄문에 너무 맹신하지는 말자ㅎ

```yaml
# 실행할 순서를 지정하는 곳이라고 보면 된다.
# test, build처럼 순서대로 작업되도록 지정한다.
# stage 이름을 같은걸 쓰는 명령어들은 병렬로 실행된다.
stages:
  - deploy

deploy-to-server:
	# 실행될 stage명
  stage: deploy 
	# 실행할 branch명, deploy branch에 파일을 올렸을때만 실행된다.
  only:
    - deploy
	# 명령어 실행전 실행되는 코드
	# 성공여부와 관련없이 실행된다.
  before_script:
    - echo 'start deployment'
    - whoami # 처음에 명령어 실행하는 user 설정 때문에 확인차 추가
	# 실제로 배포를 실행할 코드 부분
  script:
    - cd /home/ec2-user/my-project
    - git pull origin deploy
    - docker-compose stop
    - docker-compose up --build -d
    - echo 'deployment is done'
	# script 부분 실행 후 실행되는 코드
	# 성공여부와 상관없이 실행된다.
  after_script:
    - sudo docker image prune # docker-compose에 --build 옵션을 쓰면 이미지를 계속 생성해내길래, 안쓰는 image 삭제하는 명령어를 넣어봤다.
	# runner를 등록할때 입력한 tag값
  tags:
    - deploy
```

# 잘 돌아가나 확인해보기

deploy branch로 push해서 정상적으로 작동하는지 확인해보자! GitLab → CI/CD → Piplines에 들어가면 현재 상황을 체크할 수 있다!

![스크린샷 2022-05-26 오후 1.39.59.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5cc5f4d8-bb28-4890-b50b-d48d5276af8f/스크린샷_2022-05-26_오후_1.39.59.png)

위처처럼 뜨면 성공!!! 체크표시를 클릭해서 들어가면 실행된 코드를 볼 수 있다. 실패했을때 왜 실패했는지 체크가 쉬우니까 꼭 확인해보자!

# 내가 겪었던 오류들…
- Permission deny…
    처음에 gitlab을 설치할때 타 블로그를 보고 `--user=gitlab-runner`로 설정했었다. 나는 root로 설정했어야헀는데;;; 그래서 삭제한 후 `--user=root`로 재설치를 진행했다.
- 실패이유를 확인할 수가 없어…?
    ![스크린샷 2022-05-26 오후 1.44.41.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/08a79dbe-eb98-46d7-816f-4777fab0dd6c/스크린샷_2022-05-26_오후_1.44.41.png)
    처음에 push를 날려보고 실패했는데, 왜 실패했는지를 알 수가 없었다;;;;
    .gitlab-ci.yml에서 stages 부분이 없었는데, 추가하고 나니까 체크표시나 실패표시가 뜨면서 log를 볼 수 있더라.
- git pull 실패
    사실 이건 실패라고 하긴 좀 그렇고;;; 항상 ec2-user로 git pull을 했어서, root로는 권한이 없어서 생긴 문제다. 손으로 root에 접속해서 git pull을 한번 하면서 id, pw 입력한 후로를 잘 돌아갔다! (전에 한번 id, pw를 입력하면 그 정보를 저장해두는 설정을 했었다.)
