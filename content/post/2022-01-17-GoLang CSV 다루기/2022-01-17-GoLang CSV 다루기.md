---
title: GoLang CSV 다루기
description: 
date: 2022-01-17
image: 
categories:
tags:
    - GoLang
weight: 1
---

go의 csv 라이브러리를 이용해보자

## 준비

---

go의 csv는 file을 먼저 지정해주고 시작해야한다.

```go
file, err := os.Create("test.csv")
if err != nil {
	fmt.Fatalln(err)
}
```

## 읽기

---

```go
r := csv.NewReader(file)

rows, err := r.ReadAll()
if err != nil {
	log.Fatalln(err)
}

for _, row := range rows {
	fmt.Println(row)
}
```

writer는 `.Flush()`를 이용해서 사용이 끝나면 반환해줘야하지만, reader는 필요없다.

## 쓰기

---

```go
w := csv.NewWriter(file)
defer w.Flush() // 사용이 종료되면 release 시켜주는게 메모리 건강에 좋다ㅎ

headers := []string{"Link", "Title", "Location", "Salary", "Summary"}
err = w.Write(headers)
if err != nil {
	fmt.Fatalln(err)
}
```

Write는 error를 반환해주니까 error 체크를 꼭 해주자.

> go의 기본 csv 라이브러리는 GoRoutine을 지원하지 않는다;;;;
그래서 write가 많은 경우에 속도를 향상시키려면 다음과 같은 방법을 이용하자.
> 
> 1. GoRoutine을 지원하는 csv 라이브러리를 찾아서 쓴다ㅎ
> 2. WriteAll을 이용해서 한번에 쓰는 쪽으로 간다.
> 3. ~~Write마다 딜레이를 걸어주는건데...그럴거면 굳이...~~
> 