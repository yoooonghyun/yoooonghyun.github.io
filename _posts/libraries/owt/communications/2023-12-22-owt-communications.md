---
title: Communications (OWT)
author: YonghyunCho
date: 2019-12-20 14:10:00 +0800
categories: [Libraries, OWT]
tags: [owt]
render_with_liquid: false
order: 2
---

OWT (Open Webrtc Toolkit)의 내부 통신에는 process / server간의 RPC (Remote Procedure Call) 통신과, Media data교환을 위한 InternalConnection 두 가지 종류가 있다.

## RabbitMQ RPC

OWT는 기본적으로 RPC를 이용하여 process / server간 통신을 한다. 이때 RabbitMQ를 이용한 RPC를 구현했다. RPC는 server와 client로 나뉘어서 동작한다. Client에서 remote call을 하면 server에서는 그에 따른 동작과 함께 결과를 반환해야 한다. 

RPC server에서는 RPC를 받을 RPC queue를 생성하고 이를 subscribe한다. 그리고 Client는 응답을 받을 수 있는 reply queue를 생성한 후, subscribe한다. 그리고 RPC server에 remote call을 하면서 자신이 응답을 받을 수 있는 reply queue name을 알려주면, RPC server는 동작을 마친 후 reply queue에 결과를 알려준다.

Remote call을 보낼 때는 매번 correlation ID라는 unique id를 발급하여 담아보낸다. RPC server에서는 이 값을 reply에  담아 보냄으로써 어떤 호출에 대한 응답인지 식별할 수 있다.

![RPC over RabbitMQ](/assets/img/post/owt/communications/rabbitmq_rpc.png)
_Fig. 0: RPC over RabbitMQ_

## InternalConnection

OWT의 worker node간의 media data를 교환하기 위한 구현체로, In / Out 으로 방향이 나뉘어 있는 한 방향 통신이다. TCP / UDP / QUIC / SCTP를 이용하여 데이터를 교환하며, 기본 값은 TCP이다. Raw data를 다루는 VideoAgent  / AudioAgent에서는 InternalConnection를 통해 미디어 데이터를 전달하기 전에 압축을 한다.

Worker node의 createInternalConnection()을 호출함으로써 생성할 수 있으며, publish()와 subscibe() 함수를 이용하여 InternalConnection간의 연결을 맺을 수 있다. ConferenceAgent의 RoomContoller에서 connection의 생성과 연결을 관리한다. 연결의 순서는 Fig. 1를 통해 표현했다.

이때 TCP, UDP, QUIC는 in에서만 port를 열도록 구현했으며, 따라서 out에서 in으로 연결된다. 하지만 SCTP는 양방향으로 연결을 한다. 따라서 TCP, UDP, QUIC 경우 subscribe() 단계에서 InternalConnection 사이의 연결이 수행된다.

![InternalConnection_](/assets/img/post/owt/communications/internal_connection.png)
_Fig. 1: InternalConnection_

# References

- [0] [https://github.com/open-webrtc-toolkit/owt-server](https://github.com/open-webrtc-toolkit/owt-server)