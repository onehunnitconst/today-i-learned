# JWT

## JWT란?

JWT는 JSON Web Token의 줄임말이며, 당사자 간의 정보를 JSON 객체로 안전하게 전송하기 위한 독립적인 방법을 정의하는 개방형 표준([RFC 7519](https://www.rfc-editor.org/rfc/rfc7519))입니다. 이 정보는 디지털 서명으로 확인되고 신뢰할 수 있습니다.

JWT는 HMAC 알고리즘을 비롯한 비밀 키(Secret)나 RSA 혹은 ECDSA 공개 키/개인 키 쌍을 통하여 서명됩니다.

## JWT의 사용처

JWT는 다음과 같은 곳에 사용하는 것이 유용합니다.

* 인증 (Autho*rization)
    * JWT를 활용하는 가장 일반적인 시나리오입니다. 랩이오사의 서버 역시 JWT를 통하여 사용자 인증 로직을 수행합니다.
    * 사용자가 클라이언트를 통하여 로그인을 요청하여 비밀번호 혹은 다른 수단을 통해 사용자임을 인증하면, 서버는 서버의 리소스에 접근할 수 있는 권한을 담은 JWT 토큰을 발행하여 사용자에게 응답합니다. 이때 JWT 토큰은 개인 키(혹은 비밀 키(Secret))로 서명합니다.
    * 클라이언트에서는 해당 JWT 토큰을 로컬 스토리지에 저장하고, 권한이 필요한 리소스에 접근하고자 할 때 HTTP 요청에 JWT 토큰을 헤더에 포함하여 보냅니다.
    * 서버는 클라이언트의 요청에서 JWT 토큰을 공개 키로 검증하고, 요청에 따른 리소스 접근을 허용할 지 말 지 결정합니다.
* 정보 교환 (Information Exchange)
    * 공개/개인 키 쌍을 사용하여 정보에 서명할 수 있으므로 보낸 이가 누군지 확인할 수 있습니다.
    * 헤더와 페이로드를 사용하여 서명을 계산하므로 콘텐츠가 변조되지 않았음을 확인할 수 있습니다.

## JWT의 작동 방식

### JWT 토큰의 구성

JWT는 다음과 같은 3개의 파트로 나누어져 있습니다. 이는 `xxxxx.yyyyy.zzzzz` 형식의 문자열로 표현됩니다.

* x: 헤더 (Header)
* y: 페이로드 (Payload)
* z: 서명 (Signature)

### 헤더

헤더에는 2개의 정보가 담깁니다. 

* 토큰의 타입
* 토큰의 서명 알고리즘.
* 예시
    
    ```json
    {
      "alg": "HS256",
      "typ": "JWT"
    }
    ```
    

### 페이로드

페이로드는 클레임(Claim)을 포함하는 정보를 담습니다. 클레임은 사용자를 비롯하여 추가 정보를 제공하기 위하여 사용합니다.

* 등록된 클레임
    * 미리 정의되어있는 클레임이며, 일반적으로 아래의 클레임 정보가 쓰입니다.
        * **iss** (issuer, 토큰 발행인)
        * **iat** (issued at, 토큰의 발행 시간)
        * **exp** (expiration time, 토큰의 만료 시간)
        * [기타 클레임](https://tools.ietf.org/html/rfc7519#section-4.1)
    * 일반적으로 이 클레임들은 JWT를 다루는 프레임워크에서 토큰 발행 시 자동으로 페이로드에 포함하도록 구성되어 있습니다. 물론 사용자가 직접 설정할 수도 있습니다.
* 공개 클레임
    * 사용자가 임의로 지정하는 클레임입니다. 다만 [IANA 표준](https://www.iana.org/assignments/jwt/jwt.xhtml)을 지키는 것을 권고하고 있습니다.
    * 사용자 인증에 JWT를 활용하는 경우 사용자 정보를 공개 클레임에 담아 토큰을 발행합니다.
* 비공개 클레임

### 서명

서명은 BASE64로 인코딩된 헤더, 페이로드를 포함하여 헤더에 지정된 알고리즘으로 서명을 합니다.

HMAC SHA256의 경우 다음과 같은 방식으로 생성됩니다.

```json
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

### JWT 토큰

JWT 토큰의 서명이 완료되면 아래와 같은 형태의 문자열이 나옵니다.

```bash
eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.tyh-VfuzIxCyGYDlkBA7DfyjrqmSHu6pQ2hoZuFqUSLPNY2N0mpHb3nk5K17HWP_3cYHBw7AhHale5wky6-sVA
```