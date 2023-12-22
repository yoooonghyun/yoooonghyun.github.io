---
title: Control Agent
author: YonghyunCho
date: 2019-12-20 14:10:00 +0800
categories: [Libraries, OWT]
tags: [owt]
render_with_liquid: false
---

ManagementApi / Portal / ConferenceAgent는 사용자에 API를 내주거나 room, participant, stream을 관리하는 등 OWT를 제어하는 역할을 한다. ControlAgent라는 단어가 OWT에서 사용된 단어는 아니지만, 세  agent가 긴밀하게 통신하며 OWT를 제어하기 때문에 ControlAgent라는 카테고리로 분류했다.

## ManagementApi

ManagementAPI는 REST API를 노출하고 있는 WebServer의 역할을 한다. 다음의 resource에 대해 CRUD형태로 API를 제공하고 있으며, resource에 따라 일부만 제공하기도 한다.

- Rooms: POST (C), GET (R), PUT (U), DELETE (D)
- Participants: GET (R), PATCH (U), DELETE (D)
- Streams: GET (R), PATCH (U), DELETE (D)
- Streaming-ins: POST (C), DELETE (D)
- Streaming-outs: POST (C), GET (R), PATCH (U), DELETE (D)
- Recordings: POST (C), GET (R), PATCH (U), DELETE (D)
- Sipcalls: POST (C), GET (R), PATCH (U), DELETE (D)
- Analytics: POST (C), GET (R), DELETE (D)
- Token: POST (C)

API의 URI와 body에 대한 자세한 설명은 **Reference [0]**에 기술되어있다.
ManagementApi는 Express위에서 동작한다. Config로 설정되어있는 `# of CPU` 만큼의 child process를 생성하여 (cluster fork) 같은 port에 listen한다. 그리고 child process Express에 등록한 API handler를 통해 REST 요청을 처리한다. 
Room에 대한 CREATE 요청이 오면 ManagementApi는 연결된 DB (MongoDB)에 room에 대한 정보를 저장한다. 이때 저장되는 정보는 HTTP request에 의해 전달되는데, 지원되는 codec, 최대 입장 수, layout 정보 등 room의 configuration 정보이다.
하나의 room은 하나의 conference worker와 매핑 된다. 대부분의 API는 RPC를 통해서 conference worker에 위임되며, 해당 room의 conference worker가 없는 경우 새롭게 생성한다.  Conference worker는 participant, stream과 같은 context를 관리한다. 

## Portal

Signalling server의 역할을 하는 agent이다. MangementApi에서 노출하는 token을 CREATE하는 REST API를 통하여 얻은 token을 이용하여 client와 WebSocket 연결을 맺는다. WebRTC 연결을 위한 STUN 서버의 역할을 하며, client는 Portal에 publish / subscribe을 요청한다. 하지만 WebSocket call에 대한 gateway역할을 하고 대부분의 비즈니스 로직은 conference worker에서 처리다. Portal을 통해 호출하는 API는 다음과 같다. 

- login / relogin / logout: Client의 연결 및 연결  해제 (room 입장)
- text: 다른 client에 notify
- publish / unpublish: Client로부터의 stream 전달 및 취소
- stream-control: Stream의 상태 변경 (mute / mix / set region)
- subscribe / unsubscribe: Client로 stream 전달 및 취소
- subscription-control: Subscribe하는 stream의 변경
- soac: Signaling, 입력으로 넣어준 값에 해당하는 WebRTC connection을 리턴

Portal API에 대한 자세한 명세와 body Reference [1]에 명시되어있다.

## ConferenceAgent

ConferenceAgent는 conferencing을 위한 topology 연결, node 조회, context를 관리 등 대부분의 비즈니스 로직을 처리한다. 대부분의 client에서 요청은 RPC를 통해 Conference worker로 보내진다. Conference worker는 ManagementApi에 의해 room당 한 개 생성되며, conference worker에서는 room 내부에 생성된 participants / streams / subscriptions를 instance로 갖는다. 또한 RoomController / AccessController / RtcController / QuicController로 분리하여 API와 context를 관리한다.

### RoomController

InternalConnection을 연결하고 각각의 worker node에 publish / subscribe을 호출하는 등, topology를 구성 하는 로직을 담고 있다. 또한 Mixer / Transcoder를 초기화하는 역할을 한다. mix_views / terminals / streams를 instance로 관리하면 각각의 의미는 다음과 같다.

- mix_views: Audio / Video mixer의 정보
- terminals: Worker의 id / type / 연결 정보
- streams: stream의 format, owner 등에 대한 정보 (conference worker에서 관리하는 streams보다 디테일한 정보를 가지고 있음)

### AccessController

publish / subscribe로 부터 client와 연결된 worker를 생성하고, 각각의 publish / subscribe를 session (task)라는 단위로 관리한다.  내부적으로 RtcController / QuicController를 들고 있으며, WebRTC worker와 QUIC worker의 생성 /삭제에 대해서는 AccessController를 통해서 호출된다. 

### RtcController / QuicController

WebRTC worker / QUIC worker로 보내는 RPC를 호출하는 delegate.

# References
- [0] [https://software.intel.com/sites/products/documentation/webrtc/restapi/](https://software.intel.com/sites/products/documentation/webrtc/restapi/)
- [1] [https://github.com/open-webrtc-toolkit/owt-server/blob/master/doc/Client-Portal%20Protocol.md](https://github.com/open-webrtc-toolkit/owt-server/blob/master/doc/Client-Portal%20Protocol.md)