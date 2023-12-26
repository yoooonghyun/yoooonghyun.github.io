---
title: Message (SIP)
date: 2023-12-25 14:10:00 +0800
categories: [Protocols, SIP]
tags: [protocol, sip]
render_with_liquid: false
pin: true
mermaid: true
---

SIP의 메세지는 요청과 응답으로 구성되어있다. SIP 메세지의 기본적인 구성은 다음과 같다. 
- Start-line
- Message header
- Message body

## Start-line

Start-line은 어디에 어떤 메세지를 전달하는지를 표현한다. Request일때와 Response일때의 기술이 다르며 다음와 같다.

### Request-line

요청 메세지에 대한 Start-line은 다음과 같이 구성된다.

```
<Method> <Request-URI> <SIP-Version>
e.g. INVITE sip:bob@biloxi.com SIP/2.0
```

각각의 요소는 다음의 정보를 나타낸다.
- Method: 어떤 요청을 보낼지 기술
- Request-URI: 어느 UA에 요청을 보낼지 기술
- SIP-Version: UA가 사용하는 SIP version을 기술

### Status-line

응답 메세지에 대한 Start-line은 다음과 같이 구성된다. 
```
<SIP-Version> <Status-Code> <Reason-Phrase>
e.g. SIP/2.0 200 OK
```

각각의 요소는 다음의 정보를 나타낸다.
- SIP-Version: UA가 사용하는 SIP version을 기술
- Response-Code: 요청에 대한 응답 코드. HTTP Status code와 같이 숫자로 이루어져있다.
- Reason-Phrase: Status-Code에 대한 추가적인 설명.

HTTP와 마찬가지로 Status-Code는 3자리 숫자로 구성되어 있다. 그리고 가장 앞자리 숫자는 status의 category를 구분할 수 있다. 맨앞자리 숫자에 대해 구분되는 status 의 종류는 다음과 같다.

- 1xx: Provisional - 요청을 받았거나 요청을 걔속해서 처리함
- 2xx: Success - 전달받은 요청을 이해한 후 정상적으로 처리함.
- 3xx: Redirection - 요청을 처리하기 위해 추가적인 액션이 필요함.
- 4xx: Client Error - 요청에 문법적 오류가 있거나, 해당 서버의 요청으로 적합하지 않음.
- 5xx: Server Error - 정상적인 요청이지만 서버에서 처리에 실패.
- 6xx: Global Failure - 요청이 특정 서버에서 충족되지 못함.

## Message header

SIP 헤더는 HTTP와 비슷한 문법과 의미를 갖는다. 하나의 SIP 메세지에는 여러개의 헤더를 담을 수 있다.

### Header Format

헤더의 표기는 아래와 같이 colon (:)으로 이름과 값을 구분한다. header-name은 case-insensitive하여, 대소문자를 혼용하더라도 같은 이름으로 인식한다.
```
<header-name>: <header-value>
e.g. Content-Type: application/sdp
```

### Multiple Header Values

header-value가 여러개인 경우에는 아래와 같이 comma (,)를 통해서 구분한다.
```
<header-name>: <header-value1>, <header-value2>, ...
e.g. Route: <sip:alice@atlanta.com>, <sip:bob@biloxi.com>, <sip:carol@chicago.com>
```

### Header Value Parameter

header의 값에는 기본 값 뿐만 아니라 추가적인 parameter를 입력해야하는 경우도 생긴다. 이때 표기는 아래와 같이 할 수 있다.
```
<header-name>: <header-value>;<parameter-name>=<parameter-value>
e.g. Contact: <sip:alice@atlanta.com>;expires=3600
```

## Message body

Body는 요청과 응답에 모두 있을 수 있다. Body는 Content-Type 헤더에 기술되어 있는 MIME type을 따라 작성되어야한다. 또한 Content-Length 헤더에 body의 길이를 명시함으로써 데이터의 크기를 알 수 있다.

## Method Types

SIP에서 method는 다음과 같다.

- INVITE: 새로운 세션을 시작하거나 기존 세션의 파라미터를 변경할 때 사용. 가장 흔히 사용되는 메소드 중 하나로, 전화 통화나 비디오 컨퍼런스를 시작할 때 주로 사용됩니다
- ACK:INVITE 요청에 대한 최종 응답을 확인. 예를 들어, 통화가 성공적으로 설정되었을 때 ACK를 전송한다.
- BYE: 활성 세션을 종료. 예를 들어, 전화 통화를 종료할 때 BYE 메소드가 사용.
- CANCEL: 진행 중인 요청(주로 INVITE)을 취소. 예를 들어, 발신자가 통화를 시도하다가 마음을 바꿔 통화를 취소하고 싶을 때 CANCEL을 사용. 
- REGISTER: 자신의 IP주소와 포트번호를 Registrar Server에 알림.
- OPTIONS: 서버의 능력을 조사하거나, 다른 종단 간의 호환 가능한 기능을 탐색. 일반적으로 세션 시작 전에 사용.
- INFO: 세션 중에 추가 정보를 전송할 때 사용되며, 보통 제어 정보나 애플리케이션 데이터를 전달하는 데 쓰임.

## Status Codes Types

### 1xx

- 100 Trying: 요청을 다음 hop server에 전달했음.
- 180 Ringing: UA가 INVITE를 받았음.
- 181 Call Is Being Forwarded: 전화가 다른 장치로 포워딩 됨.
- 182 Queued: 통화가 일시적으로 불가능한 경우 서버가 판단하여 queueing.
- 183 Session Progress: 통화 과정중 세션 정보를 전달하기위해 사용

### 2xx

- 200: 요청을 성공함.

### 3xx

- 300 Multiple Choice: 요청에 대한 여러 가지 대안이 있음. 사용자는 이 중에서 하나를 선택해야 함.
- 301 Moved Permanently: 요청된 자원이 영구적으로 다른 주소로 이동되었음. 이후의 요청은 새 주소로 보내져야 함.
- 302 Moved Temporarily: 요청된 자원이 임시적으로 다른 주소에 있음. 이 응답은 일시적인 상황을 위해 사용되며, 원래 주소로의 복귀 가능성이 있음.
- 305 Use Proxy: 요청된 자원에 접근하기 위해 프록시 서버를 사용해야 함을 나타냄.
- 380 Alternative Service: 요청된 전화는 실패했지만, 대안이 있음. 대안은 body에 담에서 전달해야함.

### 4xx

- 400 Bad Request: 요청이 잘못되었거나 손상되었음을 나타냅니다. 문법 오류 또는 요청 처리가 불가능한 경우에 사용됩니다.
- 401 Unauthorized (인증되지 않음): 요청이 인증되지 않았음을 나타냅니다. 일반적으로 유효한 자격 증명이 요구됩니다.
- 402 Payment Required (지불 필요): 향후 사용을 위해 예약된 코드입니다. 현재 SIP에서는 사용되지 않습니다.
- 403 Forbidden (금지됨): 서버가 요청을 이해했지만, 요청을 수행할 권한이 없음을 나타냅니다.
- 404 Not Found (찾을 수 없음): 서버가 요청한 주소를 찾을 수 없음을 나타냅니다. 주소가 잘못되었거나 존재하지 않을 때 발생합니다.
- 405 Method Not Allowed (허용되지 않는 메소드): 요청된 메소드가 서버에서 지원되지 않음을 나타냅니다.
- 406 Not Acceptable (허용되지 않음): 요청된 자원이 요청자의 헤더 필드에서 정의된 조건에 맞지 않음을 나타냅니다.
- 407 Proxy Authentication Required (프록시 인증 필요): 클라이언트가 프록시 서버에서 인증을 받아야 함을 나타냅니다.
- 408 Request Timeout (요청 시간 초과): 서버가 요청을 기다리는 동안 클라이언트가 시간 내에 요청을 보내지 않았음을 나타냅니다.
- 410 Gone (사라짐): 요청한 자원이 영구적으로 사용할 수 없음을 나타냅니다. 이는 404 Not Found와 유사하지만, 여기서는 리소스가 영구적으로 사라졌음이 명확합니다.
- 413 Request Entity Too Large (요청 엔티티가 너무 큼): 요청이 너무 커서 서버가 처리할 수 없음을 나타냅니다.
- 414 Request-URI Too Long (요청 URI가 너무 김): 요청 URI가 너무 길어 서버가 처리할 수 없음을 나타냅니다.
- 415 Unsupported Media Type (지원되지 않는 미디어 타입): 요청된 메시지 포맷이 서버에서 지원되지 않음을 나타냅니다.
- 416 Unsupported URI Scheme (지원되지 않는 URI 스킴): 요청된 URI 스킴이 서버에서 지원되지 않음을 나타냅니다.
- 420 Bad Extension (잘못된 확장): 요청에 사용된 SIP 확장이 서버에서 인식되지 않음을 나타냅니다.
- 423 Interval Too Brief (간격이 너무 짧음): 요청에서 사용된 만료 시간이 너무 짧음을 나타냅니다.
- 480 Temporarily Unavailable (일시적으로 사용 불가능): 대상이 일시적으로 도달할 수 없음을 나타냅니다.
- 481 Call/Transaction Does Not Exist (통화/트랜잭션이 존재하지 않음): 요청된 통화 또는 트랜잭션이 서버에서 인식되지 않음을 나타냅니다.
- 482 Loop Detected (루프 감지됨): 요청 처리 중 루프가 감지되어 요청을 완료할 수 없음을 나타냅니다.
- 483 Too Many Hops (너무 많은 홉): 요청이 네트워크를 통과하는 동안 너무 많은 중계를 거쳤음을 나타냅니다.
- 484 Address Incomplete (주소 불완전): 요청된 주소가 완전하지 않음을 나타냅니다.
- 485 Ambiguous (모호함): 요청된 주소가 모호하여 정확한 대상을 결정할 수 없음을 나타냅니다.
- 486 Busy Here (여기는 바쁨): 요청된 대상이 현재 다른 통화 중임을 나타냅니다.
- 487 Request Terminated (요청 종료됨): 요청이 취소되었거나 다른 이유로 종료됨을 나타냅니다.
- 488 Not Acceptable Here (여기서는 허용되지 않음): 요청의 일부 파라미터가 대상에게 허용되지 않음을 나타냅니다.
- 491 Request Pending (요청 대기 중): 이전의 유사한 요청이 아직 처리 중임을 나타냅니다.
- 493 Undecipherable (해독 불가능): 요청이 암호화되었으나 서버에서 해독할 수 없음을 나타냅니다.


### 5xx

- 500 Server Internal Error: 서버에서 예상치 못한 오류가 발생함.
- 501 Not Implemented: 정상적인 요청에 대한 구현이 아직 없음.
- 502 Bad Gateway: Gateway나 proxy가 downstream server로부터 비정상적인 응답을 받음.
- 503 Service Unavailable: 서버가 일시적으로 요청을 처리할 수 없음.
- 504 Server Time-out: 서버가 요청을 외부 서버에 위임하는 경우, 외부서버의 요청으로 인해 time-out 발생.
- 505 Version Not Supported: SIP 버전이 호화되지 않음.
- 513 Message Too Large: 메세지의 크기가 처리가능한 범위를 초과.

### 6xx

- 600 Busy Everywhere: 대상이 어디에 있든 통화를 받을 수 없음. 다른 경로로의 재시도도 성공하지 못할 것임을 의미.
- 603 Decline: 사용자가 명시적으로 통화를 거절함.
- 604 Does Not Exist Anywhere: Request-URI에 해당하는 사용자 정보를 서버가 찾을 수 없음.
- 606 Not Acceptable: 요청이 서버에서 허용하는 매개변수의 범위를 벗어난 경우 발생.

## Header Types
