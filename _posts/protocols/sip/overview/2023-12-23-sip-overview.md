---
title: Overview (SIP)
author: YonghyunCho
date: 2019-12-21 14:10:00 +0800
categories: [Protocols, SIP]
tags: [protocol, sip]
render_with_liquid: false
pin: true
mermaid: true
---

## SIP (Session Initiation Protocol)

SIP (Session Initiation Protocol)는 인터넷 통신에서 음성, 비디오, 메시징 등의 멀티미디어 통신 세션을 시작, 관리, 종료하는데 사용되는 프로토콜이다. SIP는 인터넷 전화 (VoIP) 및 기타 멀티미디어 통신 서비스에서 사용자간의 연결을 위해 널리 사용된다. 


## SIP의 주요 기능

SIP의 주요 기능은 다음과 같다:

- User location: SIP는 전화번호와 유사한 SIP URI (Uniform Resource Identifier)를 사용하여 사용자를 지정한다.

- User availability: 사용자가 통화를 할 수 있는 상태인지 정의한다.

- User capabilities: 사용자가 이용할 수 있는 media 정보를 정의한다.

- Session setup: caller와 callee 양쪽에서 세션의 시작을 알리는 ringing을 수행한다.

- Session management: 데이터의 전달, 세션의 종료, 서비스 호출 등.


## SIP의 구성 요소

SIP를 구성하고 있는 주요 요소는 다음과 같다:

- User Agent (UA):
  - User Agent Client (UAC): SIP 요청을 생성하고 보내는 클라이언트 응용 프로그램이다. INVITE 메세지를 보냄으로써 세션을 시작하는 역할을 한다.
  - User Agent Server (UAS): SIP 요청을 받고 응답하는 서버 응용 프로그램입니다. UAC로 부터 전달받은 요청에 대해 적절한 응답을 

- Proxy Server:
  - SIP 요청을 중계하고, 요청을 적절한 목적지로 라우팅하는 역할을 합니다.
  - UA 간의 요청과 응답을 중계하며, 필요에 따라 요청의 일부를 수정할수도 있다.

- Redirect Server:
  - UAS에서 빌셍힌 3xx의 응답의 결과를 받아줄 서버를 의미한다.

- Registrar Server (Location server):
  - UA가 자신의 현재 위치 정보를 등록할 수 있게 해주는 서버.
  - UA가 SIP 네트워크에 로그인하고, 자신의 현재 IP 주소와 같은 위치 정보를 레지스트라에 제공한다.

- Back-to-Back User Agent, B2BUA:
  - 많은 SIP 시스템의 UA는 UAC이자 UAC로서 동작하며, 이를 B2BUA라 한다.
  - UA는 UAC로서 발급한 request에 대해서 UAS로서 논리적으로 처리할 수 있다.


## SIP의 구조

SIP는 기본적으로 유사한 메세지를 구조를 가진다. 메세지는 일반적으로 다음과 같은 요소를 포함한다.

- Start line
- Message header
- Message body

메세지는 request와 response로 나뉘며 UAC는 request를 발송하고 UAS는 전달받은 request로부터 적절한 reponse를 생성하여 전달한다. 다음은 UAC (Alice)가 UAS (Bob)에 전달하는 INVITE 메세지의 예시이다. 메세지에 대한 자세한 설명은 [SIP message](https://yooooonghyun.github.io/posts/sip-message)를 참고.


```
INVITE sip:bob@biloxi.com SIP/2.0

Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bK776asdhds
Max-Forwards: 70
To: Bob <sip:bob@biloxi.com>
From: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710@pc33.atlanta.com
CSeq: 314159 INVITE
Contact: <sip:alice@pc33.atlanta.com>
Content-Type: application/sdp
Content-Length: 142

[Body: Alice's SDP (not shown)]
```

SIP proxy는 UA간의 메세지를 중개하기 위한 역할로, UAC의 메세지를 주개하는 proxy를 outbound proxy, UAS의 메세지를 중개하는 proxy를 inbound proxy라한다. Fig. 0과 같이 UAC가 발급한 요청은 `UAC` => `Outbound Proxy` => `Inbound Proxy` => `UAS`의 경로로 전달되며, 처리된 응답은 `UAS` => `Inbound Proxy` => `Outbound Proxy` => `UAC`의 경로로 전달된다.

``` mermaid
sequenceDiagram
    participant UAC as UAC
    participant OutboundProxy as Outbound Proxy
    participant InboundProxy as Inbound Proxy
    participant UAS as UAS

    UAC->>OutboundProxy: Request
    OutboundProxy->>InboundProxy: Request
    InboundProxy->>UAS: Request
    UAS->>InboundProxy: Response
    InboundProxy->>OutboundProxy: Response
    OutboundProxy->>UAC: Response
```
<center>_Fig. 0: SIP Connection_</center>

## 세션의 생명주기


### 세션의 생성
SIP를 이용한 세션 수립은 `UAC`의 `INVITE` request로부터 시작된다. 메세지를 전달하는 proxy는 `Trying` response를 반환함으로써, 메세지가 처리되었을음 알린다. `INVITE` 메세지를 받은 `UAS`는 `Ringing` response를 반환함으로써 `INVITE` 메세지를 받았음을 알린다. 또한 `UAS`의 사용자가 전화를 수락하면 `OK` 메세지를 전달하여 `UAC`에 알린다. OK메세지를 받은 `UAC는` `ACK` request를 전달함으로써 세션이 수립된 것을 최종적으로 알린다.

### 세션의 종료
통화를 마친 사용자의 UA는 `BYE` 메세지를 발급하여 세션의 종료를 알린다. `BYE` 메세지를 받은 상대 `UA`는 `OK` 응답을 보냄으로써 세션은 종료된다.

``` mermaid
sequenceDiagram
    participant Alice as Alice's softphone
    participant AProxy as atlanta.com proxy
    participant BProxy as biloxi.com proxy
    participant Bob as Bob's SIP Phone

    Alice->>AProxy: INVITE F1
    AProxy->>BProxy: INVITE F2
    BProxy->>Bob: INVITE F4
    Bob->>BProxy: 100 Trying F5
    BProxy->>AProxy: 100 Trying F3
    AProxy->>Alice: 100 Trying F3
    Bob->>BProxy: 180 Ringing F6
    BProxy->>AProxy: 180 Ringing F7
    AProxy->>Alice: 180 Ringing F8
    Bob->>BProxy: 200 OK F9
    BProxy->>AProxy: 200 OK F10
    AProxy->>Alice: 200 OK F11
    Alice->>Bob: ACK F12
    Note over Alice, Bob: Media Session
    Bob->>Alice: BYE F13
    Alice->>Bob: 200 OK F14
```

<center>_Fig. 1: Session Establishment Process_</center>

# References

- [0] [https://datatracker.ietf.org/doc/html/rfc3261](https://datatracker.ietf.org/doc/html/rfc3261)