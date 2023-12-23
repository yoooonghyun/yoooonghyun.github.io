---
title: Overview (HTTP)
author: YonghyunCho
date: 2019-12-21 14:10:00 +0800
categories: [Protocols, HTTP]
tags: [protocol, http]
render_with_liquid: false
pin: true
---

# HTTP (Hipertext Transfer Protocol)

HTTP는 web상의 데이터 통신을 위해 hypertext를 전달하는 protocol이다.  Server-client 구조로 동작하며, Application layer에 속한다. 일반적으로 client side에서 요청을 보내면 server side에서 응답을 보내는 형태로 동작한다.


![HTTP-Based System](/assets/img/post/protocol/http/overview/http-based-system.png)
_Fig. 0: HTTP-Based System [0]_

## HTTP 0.9 (1991)

HTTP의 prototype버전으로 정식 출시 전이기 때문에 0.9 버전을 붙임.

### Method

GET method만 지원

### Header

헤더가 존재하지 않았으며, 따라서 HTML 파일만 전달

## HTTP 1.0 (1996)

정식으로 출시된 HTTP의 첫 버전으로, 버전 정보가 요청에 전송되기 시작.
### Method

GET/HEAD/POST 제공

### Status Code

status code의 도입
- 2xx: 200, 201, 202, 204
- 3xx: 300, 301, 302, 303, 304
- 4xx: 400, 401, 403, 404
- 5xx: 500, 501, 502, 503

### Header

Header의 도입. Header는 요청과 응답에 모두 도입 됐으며, 공통/요청/응답 헤더로 나뉜다.
content-type 헤더의 도입으로, HTML 이외의 다른 데이터를 정송하는 기능이 추가되었다.

## HTTP 1.1 (1997)

HTTP 1.0의 모호함을 명확하게 하고 많은 개선사항을 도입.

### Connection

Connection을 재사용할 수 있게 하여 단일 문서 안에 임베딩된 resource 불러오는데 시간을 절약.

- Connection: keep-alive

Pipelining을 통해 첫 번째 요청이 완료되기 전에 다른 요청을 보낼 수 있도록 개선. (서로 다른 connection)

### Encoding

Chunked-Transfer-Encoding을 통해 파일에 대한 빠른 응답 지원.
언어, 인코딩 혹은 타입을 통한 컨텐츠 협상의 도입.

### Cache
- Cache-control
  - no-store: 캐시하지 않음
  - no-cache: 캐시하지만 서버에유효성 검증
  - private: 단일사용자를 위한 캐시로, 공유 캐시에 저장 X
  - public: 공유 캐시에 저장 가능
  - max-age: 만료 시간
  - must-revalidate: 캐시 만료 후 서버에 유효성 검증
- Etag를 통한 유효성 검증

### Host

Host 헤더를 통해 같은 IP지만 다른 Domain을 통해 호스팅가능

## HTTP 2 (2015)

SPDY안에 구현되었던 내용들이 HTTP 표준으로 편입

### Frame/Message

기존 HTTP의 최소 단위인 Message보다  더 작은 단위인  Frame을 지원. Frame은 단위의 데이터 전송은 HTTP의 더 유연한 동작을 가능하게 함.

### Frame type

- DATA: Body 데이터의 전달
- HEADER: 헤더 영역 전달 시작
- CONTINUATION: 헤더 전달 후 
- PUSH_PROMISE: Server push 기능
- GOAWAY: Connection의 종료

![Message / Frame of HTTP](/assets/img/post/protocol/http/overview/message-frame.png)
_Fig. 1: Message / Frame of HTTP [1]_

### Connection/Stream

연속된 Frame의 양방향 전달 채널인 Stream의 도입. 하나의 connection에서 여러 개의  Stream을 생성하며 이를 통해 mutiplexing 제공. Frame에는 Stream id를 기록하는 필드가 있어, 동일 Stream에 연속된 Frame을 구분하여 전달. Stream에는 priority와 서로간의 dependency를 설정할 수 있으며, 이를 통해 concurrency control 동작.

### Header Compression

HPACK 헤더 압축을 통한 오버헤드 감소. Cookie 별도로 처리.

### HPACK Algorithm

- Huffman coding: 빈도수에 따라 다른 비트 할당
- Static table: 자주 쓰는 header 미리 할당
- Dynamic table: static table에 없는 header 저장

![Static Table in HPACK](/assets/img/post/protocol/http/overview/hpack-static-table.png)
_Fig. 2: Static Table in HPACK [6]_

### Secure

Protocol 내에 TLS 1.2를 이용한 암호화를 기본으로 지정.  Upgrade/Alt-Svc 헤더를 통해서 clear text over http2 연결로 변경 가능.
- Upgrade: h2c
- Alt-Svc: h2c

### Server Push

PUSH_PROMISE Frame을 이용하여, client의 요청 없이 서버에서 전송하는 server push 지원. 여러 리소스를 한번에 다운로드 하는 경우 Bandwidth/latency를 줄일 수 있음.

## HTTP 3 (2022)

### QUIC Transport

기존 TCP/IP 위에 TLS를 이용한 암호화 방식인 HTTP대신 UDP 위에 DTLS를 올린  통신 방식. Multiplexing 지원을 통해 HTTP 안에서 발생하는 head-of-line bloking은 해결되었으나, TCP 레이어에서 발생하는 문제는 해결 불가능했다. 하지만 UDP기반의 통신을 하지만 신뢰성은 software 레벨에서 보장하여 이를 해결했다.

![HTTP Network Layers](/assets/img/post/protocol/http/overview/http-layer.png)
_Fig. 3: HTTP Network Layers_

### Connection

QUIC Transport의 또다른 특징은 connection이 간결한 커넥션 과정이다. 기존 TLS v1.2를 사용하는 HTTP2는 연결과정을 수행하는데 총 3 RTT가 필요하다.
TLS v1.3이 지원되면서 HTTPS 연결은 2.5 RTT로 줄었으며, client에서 TLS Finished를 보내면서 동시에 요청을 보냄으로써 server의 최초 응답을 받기까지 1 RTT 만큼의 통신을 감소시켰다.

![Handshake Processes of HTTP2](/assets/img/post/protocol/http/overview/http2-handshake.png)
_Fig. 4: Handshake Processes of HTTP2_

QUIC에서는 UDP를 사용함으로써 TCP에서 필요한 3 way handshake 과정이 없어졌다. 따라서 연결 과정에서 1 RTT만큼의 이득을 볼 수 있다. Client가 이미 교환된 key를 클라이언트가 가지고 있고, 이를 재연결시에 사용한다면 TLS v1.3의 handshake는 1 RTT 까지 줄어들 수 있다. 이때 handshake과정에서 HTTP payload를 전달함으로써 handshake 과정은 0 RTT로 수렴한다.

![Handshake Processes of HTTP3](/assets/img/post/protocol/http/overview/http3-handshake.png)
_Fig. 5: Handshake Processes of HTTP3_

# References

- [0] [https://developer.mozilla.org/ko/docs/Web/HTTP/Overview](https://developer.mozilla.org/ko/docs/Web/HTTP/Overview)
- [1] [https://developer.mozilla.org/ko/docs/Web/HTTP/Messages](https://developer.mozilla.org/ko/docs/Web/HTTP/Messages)
- [2] [https://www.rfc-editor.org/rfc/rfc1945](https://www.rfc-editor.org/rfc/rfc1945)
- [3] [https://www.rfc-editor.org/rfc/rfc2616](https://www.rfc-editor.org/rfc/rfc2616)
- [4] [https://www.rfc-editor.org/rfc/rfc7540](https://www.rfc-editor.org/rfc/rfc7540)
- [5] [https://www.rfc-editor.org/rfc/rfc9114.html](https://www.rfc-editor.org/rfc/rfc9114.html)
- [6] [https://www.rfc-editor.org/rfc/rfc7541](https://www.rfc-editor.org/rfc/rfc7541)