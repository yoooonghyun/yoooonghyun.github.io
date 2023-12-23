---
title: Overview (OWT)
author: YonghyunCho
date: 2019-12-20 14:10:00 +0800
categories: [Libraries, OWT]
tags: [owt]
render_with_liquid: false
order: 1
---

# OWT (Open Webrtc Toolkit)

OWT는 Webrtc 기반의 conference와 stream을 지원하는 media server이다. SFU (Selective Forwarding Unit)와 MCU (Multi-point Control Unit)방식을 혼용하고 있으며, scalable한 conferencing solution이다.
Node js기반의 코드이지만 C++을 기반으로 구현한 media pipeline을 add-on 시켰다. Pipeline의 multimedia processing에는 Webrtc와 FFmpeg을 이용했다. 지원하는 specification은 다음과 같다.

- Video  codecs: VP8, VP9, H264, H265, 
- Audio codecs: G711, G722, iSAC, OPUS, AAC
- Protocols: WebRTC, RTSP, RTMP, HLS, MPEG-DASH

## Architecture

OWT는 ManagementApi, ClusterManager, ConferenceAgent, Portal, WorkerAgent와 같은 서버를 여러 개 띄우면서 동작한다. 서버는 서로 RabbinMQ를 기반의 RPC 통신을 이용하여 통신하며, 각각의 역할은 다음과 같다.

- MangementApi: Rest api server
- ClusterManger: WokerAgent의 scheduling
- ConferenceAgent: Room, participant, stream과 같은 state관리
- Portal: Signaling server
- WorkerAgent: Pipeline을  구성하는 worker (node)들을 관리하는 agent

WorkerAgent의 역할로는 streaming gateway / media processing / recording로 나뉘며 각각의 역할에는 다음과 같은 agent가 있다.

- Streaming gateway: QuicAgent, SipAgent, WebrtcAgent, StreamingAgent
- Media processing: VideoAgent, AudioAgent
- Recording: RecordingAgent

Agent는 ClusterManger에 의해 scheduling되는 cluster worker의 성격을 가짐과 동시에 실제  worker node를 생성하는 node manger의 성격을 지닌다. 각각의 worker node는 실제로 미디어 데이터를 처리하는 프로세스를 의미한다. 

Worker node는 InternalConnection이라는 구현체를 이용하여 서로 미디어 데이터를 교환할 수 있으며, 이 경우 한다. InternalConnection은 InternalIn / InternalOut으로 구성되어 있으며, 서로 다른 worker node의 InternalOut과 InternalIn이 connection을 맺은 후, out에서 in 방향으로 데이터를 전달한다. 이때 TCP, UDP, SCTP, QUIC중 하나의 transport layer 이용하여 미디어 데이터를 교환한다.

## Example

![Overall Architecture of OWT Server](/assets/img/post/owt/overview/architecture.webp)
_Overall Architecture of OWT Server_

References
- [0] [https://github.com/open-webrtc-toolkit/owt-server](https://github.com/open-webrtc-toolkit/owt-server)
- [1] [https://software.intel.com/sites/products/documentation/webrtc/restapi/index.html](https://software.intel.com/sites/products/documentation/webrtc/restapi/index.html)