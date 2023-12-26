---
title: Conferencing
date: 2019-12-21 14:10:00 +0800
categories: [Multimedia, ETC]
tags: [multimedia]
render_with_liquid: false
---

웹 브라우저상의 화상통화는 WebRTC를 이용하여 구현할 수 있다. WebRTC는 peer간의 미디어 데이터를 교환하는 프로토콜로, 양 client의 P2P연결을 통해서 동작한다. 따라서 화상회의의 참여자가 증가함에 따라 P2P 연결이 많아지면, client의 부하 (network, computation power)로 이어진다. Client의 부하를 해결하기 위해, 중간에 미디어서버를 두고 미디어 서버와 client간의 데이터를 relaying하거나 (SFU), 심지어는 서버에서 믹싱과정을 거친 데이터를 전달하기도 한다. (MCU)


![Types of Conferencing](/assets/img/post/multimedia_etc/conferencing/conferencing-types.png)
_Fig. 0: Types of Conferencing_

## Mesh Networking

Mesh Networking 방식은 모든 참여자가 P2P연결을 맺는 방식으로, 미디어 서버 없이 구현할 수 있는 방식이다. 따라서 N명의 참여자가 존재한다면, 소모되는 네트워크의 양은 N배가 된다. 많은 네트워크 비용이 필요한 미디어 데이터의 특성상, 참여자가 증가함에 따라 client에 큰 부담이 된다. 그럼에도 구현이 간단하다는 장점이 있기 때문에 1:1 혹은 소규모 화상회의를 빠르게 구현하기에 적합한 방법이다.

## SFU

SFU 방식은 client간에 미디어 데이터를 relaying 서버를 두고 데이터를 교환하는 방식이다. Client는 자신의 upstream data를 서버에 한번만 보내면 되지만, 다른 참여자의 수 만큼 서버와 연결을 맺고 각 스트림 데이터를 받는다. Relaying 서버는 별도의 미디어 처리 없이, SRTP/RTCP 데이터를 각 사용자에게 전달한다. 하지만 각 connection별로 DTLS로 데이터가 암호화되는 WebRTC의 특성상, decryption/encryption과정을 거친다. 비교적 서버 부담도 적고, client의 부담도 줄어드는 방식. Janus/Kurento와 같은 open source 프로젝트가 SFU를 지원한다.

## MCU

MCU 방식은 각 참여자의 미디어 데이터를 서버에 모아 믹싱한 후, 그 결과물을 다시 각각의 참여자들에게 전달한다. 따라서 client는 자신의 미디어 데이터만 서버로 전달하면 되고, 믹싱된 하나의 화면만 받아오기 때문에 client의 부하는 굉장히 줄어든다. 하지만 그만큼의 연산을 서버에서 하기 때문에 높은 computing power가 필요하다. 아주 열악한 환경의 사용자를 커버해야하거나 모든 사용자가 동일한 view를 본다면 고려해볼 수 있는 방식이다.

### Optimization Point

![Multimedia Pipeline of Conferencing](/assets/img/post/multimedia_etc/conferencing/conferencing-processing.png)
_Fig. 1: Multimedia Pipeline of Conferencing_

화상통화는 위 그림과 같은 일련의 과정을 통해서 이루어진다. 이때 가장 병목이 될 수 있는 과정은  Video processing과정과 peer간의 data transport 과정이다.

- Video Encoding/Decoding
  - 일련의 화상회의 과정에서 가장높은 computation power를 요구
- Peer간 data transport
  - Network 상태에 따라 병목지점

![Multimedia Pipeline of Conferencing w/ Mutiple Tx](/assets/img/post/multimedia_etc/conferencing/multi-tx-processing.png)
_Fig. 2: Multimedia Pipeline of Conferencing w/ Mutiple Tx_

화상회의의 참여자에 비례해서 Rx에서는 더 높은 computing자원을 필요로한다. 심지어 client의 상황에따라 추가적인 mixer가 필요할 수 있다. 만약 Mesh를 통한 화상회의를 구축했다면, Tx쪽에서도 참여자에 비례해서 encoder와RTC Sender가 필요하다. WebRTC에서 multicast를 지원하지 않기 때문에  client side의 최적화 여지는 적은편이다.

병목이 되는 video processing/peer간의 data transport는 video 해상도에 의해 많은 영향을 미친다. 해상도가 낮으면 압축 과정의 연산량도 적으며, 결과물 또한 작은 용량을 차지한다. 따라서 미디어 데이터 전송을 위한 network bandwidth도 줄어든다. peer의 상황에 따라서 작은 사이즈의 이미지가 필요하다면, negotiation 단계 혹은 streaming중에 낮은 해상도의 화면으로 변환하여 전송하는 방법이 가장 현실적인 최적화 방법이다.