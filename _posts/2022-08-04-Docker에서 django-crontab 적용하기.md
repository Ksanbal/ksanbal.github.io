---
layout: post
---
프로젝트를 진행하다보면 심심치않게 스케줄러를 개발해야하는 경우가 생긴다. 리눅스에 있는 crontab이라는 스케줄러를 이용해서 개발할 수 있는데, 최근 프로젝트에서 적용하는데 꽤나 고생해서 그 내용을 정리할 생각이다. django-crontab 자체를 다룬다기보다는 docker랑 같이 사용방법에 대해서 위주로 다룰 예정!

> **개발환경**
MacOS M1
Docker, Docker-compose
Django
Firebase
> 

# django-crontab

Django에서 crontab을 적용하고 실행시켜주는 패키지이다. 기본적인 사용법은 간단하고, 공식문서와 블로그들이 괜찮아서 시작부분은 빠르게 패스!

> 공식문서
> 
> 
> [django-crontab](https://pypi.org/project/django-crontab/)
> 

## Docker랑 합체

crontab은 linux의 기본 패키지이지만, docker에서 python 이미지를 불러와서 다른 패키지들(vi 같은 기본적인 것들…)과 마찬가지로 만들때는 포함되어있지 않다. 따라서 `dockerfile`에서 따로 설치하고 적용시켜줘야한다. 나는 `docker-entrypoint.sh`를 같이 쓰고 있기 때문에 참고….

```docker
# dockerfile
...
ADD ./docker-entrypoint.sh ./

RUN chmod +x ./docker-entrypoint.sh
ENTRYPOINT ./docker-entrypoint.sh
...
```

```bash
# docker-entrypoint.sh
...
echo "3-1. Update apt-get"
apt-get update

echo "3-2. Install cron"
apt-get install -y cron

# cron에 대한 log를 남기기 위해 디렉토리와 파일을 미리 만들어놓는다.
echo "3-2. Create cron.log"
mkdir /usr/log
touch /usr/log/cron.log

# django-crontab add 명령어
echo "3-3. Add CRON"
python manage.py crontab add

echo "3-4. Show CRON"
python manage.py crontab show
...
```

이렇게 하면 끝!!

라고 하기엔 내가 너무 많은 고생을 했기에, 그 이후의 작업이 있다.

## crontab 자체가 안돌아가는데…?

여러 블로그를 참고하다보니, crontab 설치는 했는데 crontab 실행은 안해놓은 케이스… `docker-entrypoint.sh`에 실행코드를 추가해주자

```bash
# docker-entrypoint.sh
...
echo "3-1. Update apt-get"
apt-get update

echo "3-2. Install cron"
apt-get install -y cron

# cron에 대한 log를 남기기 위해 디렉토리와 파일을 미리 만들어놓는다.
echo "3-2. Create cron.log"
mkdir /usr/log
touch /usr/log/cron.log

# linux에서 cron을 service로 실행시켜준다.
echo "3-3. Start cron"
service cron start
service cron status

# django-crontab add 명령어
echo "3-4. Add CRON"
python manage.py crontab add

echo "3-5. Show CRON"
python manage.py crontab show
...
```

## crontab은 도는데, 왜 아무일도 없음…?

django-crontab은 django 환경에서 코드를 실행시켜준다. 이 말은 django에서 설정한 DB에 접근할 수 있다는 이야기이다. 사실 우리가 스케줄러를 만드는 이유가 주기적으로 DB의 데이터를 활용하기 위함이니까, 굉장히 중요한 부분이라고 할 수 있다. 그런데 여기서 문제는, django-crontab은 linux의 cron을 활용하기 때문에 crontab 자체의 특성을 그대로 가진다. docker안에서 cron이 어떻게 등록됬는지 확인하기위해, container에 접속한 후 `crontab -l`로 확인해보자

```bash
# crontab -l
*/10 * * * * /usr/local/bin/python /usr/src/manage.py crontab run [CRONJOB ID] >> /usr/log/cron.log # django-cronjobs for [PROJECT NAME]
```

등록된 코드를 보면 정상적으로 안돌아갈 이유가 없어보이지만, 여기서 **주의할 점은 linux의 crontab은 환경변수를 공유하지 않는다는 것이다;;;;** log를 열심히 찍으면서 확인해본 결과 django 코드에서 os.getenv()에서 가져오는 환경변수들을 cron에서는 가져오지 못하는 것이다. DB정보나 Firebase 같은 정보들은 전부 .env에 저장해서 따로 관리했는데, 그럼 어떻게 해야할까. 

## python-dotenv 패키지 사용하기

사실 어차피 private 레파지토리니까 그냥 코드에 적어버릴까…했지만 그건 좀 선 넘은 것 같아서, python-dotenv라는 패키지를 찾아서 해결하기로 했다. 환경변수를 사용하는 모든 곳에 적용해도 좋지만, crontab이 사용하는 부분에서만 써도 괜찮을 것 같아서, 특정 부분에만 적용했다.

> 패키지
> 
> 
> [python-dotenv](https://pypi.org/project/python-dotenv/)
> 

```bash
# python-dotenv 설치
poetry add python-dotenv
# or pip install python-dotenv
```

```python
# settings.py
...
from dotenv import dotenv_values

BASE_DIR = Path(__file__).resolve().parent.parent
ENV_FILE = str(Path(BASE_DIR.parent)) + "/.env"
myEnvs = dotenv_values(ENV_FILE)
...
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "HOST": myEnvs["DB_HOST"],
        "PORT": myEnvs["DB_PORT"],
        "USER": myEnvs["DB_USERNAME"],
        "PASSWORD": myEnvs["DB_PASSWORD"],
        "NAME": myEnvs["DB_NAME"],
    },
}
...
# Cron
CRONJOBS = [
    ("*/10 * * * *", "radi.cron.test", ">> /usr/log/cron.log"),
]
CRONTAB_COMMAND_PREFIX = f"GOOGLE_APPLICATION_CREDENTIALS={myEnvs['GOOGLE_APPLICATION_CREDENTIALS']}"
```

이 외에도 firebase 사용하는 부분 등에서 적용해줬고, 드디어 정상적으로 실행되는 것을 확인할 수 있었다. 추가로 따로 환경변수를 적용해주고 싶은 내용은 CRONTAB_COMMAND_PREFIX으로 세팅해줄 수 있다. Firebase의 경우 key파일의 경로를 코드에서 가져다쓰는게 아닌 환경변수에 담아놔야해서 추가로 따로 적용해줬다.

나의 하루…불태웠다…🔥

> **참고**
> [https://github.com/kraiz/django-crontab/issues/88](https://github.com/kraiz/django-crontab/issues/88)
> [django-crontab](https://pypi.org/project/django-crontab/)