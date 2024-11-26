---
title: NestJS으로  AES Encryption & Decryption(암호화&복호화)
description: 
date: 2022-11-17
image: 
categories:
    - NestJS
tags:
weight: 1
---

User 정보를 다루다보면 개인정보들은 암호화해야하는 경우가 있다. 비밀번호는 당연히 hashing 처리해오다 문득 이메일, 전화번호 같은 정보들도 암호화가 필요하다는 생각이 들었다. 그런데 문제는 비밀번호와 달리 복호화도 가능해야한다는 것! NestJS에서는 공식문서에서 [Encryption과 Hashing](https://docs.nestjs.com/security/encryption-and-hashing)에 대한 내용을 다루고 있다. 공식문서 내용과 살짝 커스텀한 내용으로 정리해보자.

## AES()가 뭐죠...?
AES(Advanced Encryption Standard)는 동일한 키를 가지고 암호화, 복호화를 진행하는 **대칭 키 알고리즘**이다. DES에 비해 빠르고 코드 효율성이 높다고 한다. 자세한 내용은 아래의 링크를 참조하면 좋을 것 같다.
[[AES 암호 알고리즘(Advanced Encryption Standard)](https://www.crocus.co.kr/1230)](https://www.crocus.co.kr/1230)

## MySQL은 AES 암호화를 지원합니다.
MySQL 5 이상부터 AES 암호화, 복호화를 지원한다고 한다. 그래서 QUERY에서 암호화를 적용해서 읽고 쓰고가 가능하다. 나는 주로 PostgreSQL을 사용하는데, extension 설치를 통해서 적용 할 수 있다고 한다. 하지만 나는 TypeORM을 쓰고 있기 때문에, 따로 QUERY로 코드를 변경하는게 썩 마음에 들지 않아서 서버에서 암호화 복호화 하는 방식을 선택했다.

## 소스코드
일단은 공식문서에서 제공하는 코드부터 살펴보자

### 암호화
```typescript
import { createCipheriv, randomBytes, scrypt } from 'crypto';
import { promisify } from 'util';

const iv = randomBytes(16);
const password = 'Password used to generate key';

// The key length is dependent on the algorithm.
// In this case for aes256, it is 32 bytes.
const key = (await promisify(scrypt)(password, 'salt', 32)) as Buffer;
const cipher = createCipheriv('aes-256-ctr', key, iv);

const textToEncrypt = 'Nest';
const encryptedText = Buffer.concat([
  cipher.update(textToEncrypt),
  cipher.final(),
]);
```

- iv (Initalization Vector) : 복호화 키 유추를 막기위해 사용하는 완전 랜덤한 값이다. 랜덤값을 사용하는게 권장되지만 데이터 저장시 사용한 iv 값을 같이 저장하기 때문에(ex) {iv}:{encryptedData} ) 데이터 크기가 커진다는 단점이 있다. 또한 암호화된 값이 매번 달라지는게 장점이면서도, unique 체크를 해야하는 데이터에 경우엔 맞지않는다. AES는 128 bit(16 byte) 단위로 암호화하기 때문에 iv도 16 byte로 설정해야한다.
- password : 암호화에 사용될 키 값을 생성하기 위한 비밀번호이다. 절대 유출되지 않아야하기 때문에, .env를 사용해서 설정하자.

### 복호화
```typescript
import { createDecipheriv } from 'crypto';

const decipher = createDecipheriv('aes-256-ctr', key, iv);
const decryptedText = Buffer.concat([
  decipher.update(encryptedText),
  decipher.final(),
]);
```

공식문서에서의 코드로 실행해보면 결과값이 Buffer 값으로 나온다. DB에는 Buffer값을 저장할 순 없으니, base64 또는 hex로 변환해서 사용하는게 좋은 것 같다. 그래서 공식문서의 코드를 바로 사용하기엔 조금 애매하고 실환경에 맞게 다시 코드를 짜보자

### 1. iv값을 랜덤으로 설정하는 경우
- 불러오기 편하게 repository 형식으로 service를 만들어주자.
- Key는 미리 지정한 비밀번호를 불러와서 생성한다. Key 자체를 만들어서 보관해도 괜찮을 것 같다.
- 암호화된 값을 hex로 관리하는 경우도 있던데 나는 base64로 처리해봤다.
- 혹시몰라 암호화 방식도 .env로 지정해줬다. 공식문서의 aes-256-ctr 외에도 많은 알고리즘이 있으니, 그 중에서 선택해서 사용하면 좋을 것 같다.

```typescript
import { promisify } from 'util';
import { scrypt, createCipheriv, createDecipheriv, randomBytes } from 'crypto';
import { Injectable } from '@nestjs/common';

@Injectable()
export class EncryptionService {
  /**
   * AES 암호화, 복호화용 KEY값 반환
   * @returns key: Buffer
   */
  private async getKey(): Promise<Buffer> {
    return (await promisify(scrypt)(
      process.env.AES_ENCRYPTION_PASSWORD,
      'salt',
      32,
    )) as Buffer;
  }

  /**
   * AES-256 암호화
   * @param text 암호화할 텍스트
   * @returns 암호화된 base64 텍스트
   */
  async encrypt(text: string): Promise<string> {
    const iv = randomBytes(16);

    const key = await this.getKey();
    const cipher = createCipheriv(process.env.AES_ALGORITHM, key, iv);

    const encryptedText = Buffer.concat([cipher.update(text), cipher.final()]);

    return iv.toString('base64') + ':' + encryptedText.toString('base64');
  }

  /**
   * AES-256 복호화
   * @param text 복호화할 텍스트
   * @returns 복호화된 텍스트
   */
  async decrypt(text: string): Promise<string> {
	const splitText = text.split(':')
    const iv = Buffer.from(splitText[0], 'base64')
    const encryptedText = Buffer.from(splitText[1], 'base64');

    const key = await this.getKey();

    const decipher = createDecipheriv(process.env.AES_ALGORITHM, key, iv);
    const decyptedText = Buffer.concat([
      decipher.update(encryptedText),
      decipher.final(),
    ]);

    return decyptedText.toString();
  }
}
```

위 방식으로 진행하면 iv가 항상 변하기 때문에 암호화된 값도 항상 변하게 된다. 하지만 일반적인 데이터라면 상관없지만 이메일, 전화번호, 주민번호 처럼 DB에서 unique한 속성을 가져야하는 경우가 있다. 그런 경우엔 위의 코드를 적용할 수 없다.

### 2. iv값을 지정된 값으로 설정하는 경우
- iv 값도 .env에 지정해서 적용해주자.

```typescript
import { promisify } from 'util';
import { scrypt, createCipheriv, createDecipheriv } from 'crypto';
import { Injectable } from '@nestjs/common';

@Injectable()
export class EncryptionService {
  /**
   * AES 암호화, 복호화용 KEY값 반환
   * @returns key: Buffer
   */
  private async getKey(): Promise<Buffer> {
    return (await promisify(scrypt)(
      process.env.AES_ENCRYPTION_PASSWORD,
      'salt',
      32,
    )) as Buffer;
  }

  /**
   * AES-256 암호화
   * @param text 암호화할 텍스트
   * @returns 암호화된 base64 텍스트
   */
  async encrypt(text: string): Promise<string> {
    const iv = Buffer.from(process.env.AES_ENCRYPTION_IV, 'base64');

    const key = await this.getKey();
    const cipher = createCipheriv(process.env.AES_ALGORITHM, key, iv);

    const encryptedText = Buffer.concat([cipher.update(text), cipher.final()]);

    return encryptedText.toString('base64');
  }

  /**
   * AES-256 복호화
   * @param text 복호화할 텍스트
   * @returns 복호화된 텍스트
   */
  async decrypt(text: string): Promise<string> {
    const iv = Buffer.from(process.env.AES_ENCRYPTION_IV, 'base64');
    const encryptedText = Buffer.from(text, 'base64');

    const key = await this.getKey();

    const decipher = createDecipheriv(process.env.AES_ALGORITHM, key, iv);
    const decyptedText = Buffer.concat([
      decipher.update(encryptedText),
      decipher.final(),
    ]);

    return decyptedText.toString();
  }
}
```

## 실제로 서비스할때는 어떻게 운영할까
DB에는 이제 암호화된 값으로 저장되어 있기 때문에 바로 SELECT 해서 return 해주면 암호화된 값을 내려보내게된다. 그래서 복호화를 해줘야 의미있는 데이터가 되는건데, 복호화는 어디서 진행하는게 맞을까.

### Client에서 복호화
- key, iv값을 client와 공유해서 client에서 직접 복호화 할 수 있다.
- http 통신중에 해킹 당할 것을 생각한다면, 위 방식이 맞을 것 같다.
- 하지만 Client의 컴퓨팅 파워를 사용해야하고, key 또는 iv 값이 변경되면 해당 내용이 반영되어야한다.
- iv는 유츌되도 상관없지만 key은 절대 안된다. Mobile이 비교적 이런 이슈에 취약하다고 느껴지는데, Android는 Proguard 설정으로 다시 한번 암호화된 소스코드를 가지고 있으니 괜찮을 것 같기도 하고...React와 같은 Web Frontend에서는 어떻게 할 수 있을지 모르겠다.

### Backend에서 복호화
- 직접 복호화를 해서 데이터를 전달해 줄 수 있다.
- 결국 DB에만 암호화되고, 통신중에는 암호화된 값이 아니기 때문에 암호화가 크게 의미가 없는 듯 있는 듯한 느낌.
- Key 또는 iv값 변경에도 크게 영향 받을 일 없고, 서버가 털리지 않는 이상 key가 유출될 위험도 적다.

### 그래서 어떻게 할까요?
모바일 개발자분과 다른 서버 개발자분과 함께 얘기해봤었는데, 결론은 **정답이 없다**였다. 위의 설명한 장단점에 따라 상황별로 선택하는게 맞을 것 같고, Backend 또는 Frontend 중 입김이 샌 쪽이 하자는대로 될 것 같기도 하다ㅎㅎ

# 정리
평소 비밀번호 외에는 크게 암호화에 대해 관심이 없었는데, 이번 프로젝트에서 개인정보 관련된 보안을 적용하면서 배우게 되었다. 정책적인 부분이지만 개발자도 알고 처리하는게 필요하다고 느껴진다. 개인정보는 대상을 특정할 수 있는 데이터를 의미한다. 어디까지 암호화 할 것 인가도 고민이 있었는데, 법으로 지정된 부분을 참고하면 좋을 것 같다. 
[암호화 해야 하는 정보와 저장하면 안 되는 정보](https://library.gabia.com/contents/infrahosting/2665/)