---
title: Python dictionary로 Class instance 생성하기
description: 
date: 2021-09-10
image: 
categories:
tags:
weight: 1
---

FastAPI 개발하다 requst를 json으로 받았는데, 그걸 Model에 저장하려고 보니까 Column 수가 많아지면 하나하나 매칭시켜주는게 너무 무지성인 것 같아서 dict을 자동으로 넣어주는 코드를 찾아봤다.

모델 Class를 이렇게 작성하면

```python
class Weather(Base):
    __tablename__ = "weathers"
    
    id = Column(Integer, primary_key=True, index=True)
    region_name = Column(String(20), comment="지역명")
    latitude = Column(Float, comment="위도")
    longitude = Column(Float, comment="경도")
    update_time = Column(DateTime, comment="마지막 업데이트일", nullable=True)
    weather_json = Column(JSON, comment="날씨 데이터", nullable=True)
```

아래와 같은 방법으로 DB에 저장할 수 있는데

```python
# request를 dict으로 받으면
weather = models.Weather(
		region_name = request['region_name'],
		latitude = request['latitude'],
		longitude = request['longitude'],
		update_time = request['update_time'],
		weather_json = request['weather_json'],
)
db.add(weather)
db.commit()
```

이게 Column이 많아지면 instance 생성할때마다 하나하나 넣어주거나 `__init__`에서 미리 지정해줘야한다. 하지만 그건 너무 귀찮고 무지성인걸...?

그래서 찾아보니까

```python
class Weather(Base):
    __tablename__ = "weathers"
    
    id = Column(Integer, primary_key=True, index=True)
    region_name = Column(String(20), comment="지역명")
    latitude = Column(Float, comment="위도")
    longitude = Column(Float, comment="경도")
    update_time = Column(DateTime, comment="마지막 업데이트일", nullable=True)
    weather_json = Column(JSON, comment="날씨 데이터", nullable=True)

		def __init__(self, dictionary: dict):
        for k, v in dictionary.items():
            setattr(self, k, v)
```

```python
weather = models.Weather(request)
db.add(weather)
db.commit()
```

이렇게 써주면 instance 생성할때 dict의 key value로 맞춰서 생성된다.

`setattr()`을 이렇게 쓸줄은 몰랐지...