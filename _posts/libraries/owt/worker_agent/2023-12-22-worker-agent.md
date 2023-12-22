---
title: Worker Agent
author: YonghyunCho
date: 2019-12-20 14:10:00 +0800
categories: [Libraries, OWT]
tags: [owt]
render_with_liquid: false
---

WorkerAgent는 OWT의 meida processing을 담당하는 WorkerNode을 관리하는 역할을 함과 동시에, ClusterManger의 관리를 받는 ClusterWorker로써 동작한다.

## ClusterWorker

WorkerAgent는 생성과 동시에 RpcClient를 생성한다. 이후에 ClusterManger에 자신을 등록하며, 등록을 성공하면 주기적으로 ClusterManager에 heath check를 위한 RPC를 보낸다. 또한, WorkerNode의 생성 / 소멸에 따라 ClusterManger에 RPC를 전달한다. 전달하는 RPC의 종류에는 reportLoad / pickUpTasks / layDownTask / unschecdule / quit / reportState / join / keepAlive가 있으며, 다음과 같은 상황에서 전달한다.

- reportLoad: 주기적으로 resource 사용 양을 보고
- pickUpTasks: Task의 추
- layDownTask: Task 제거
- unschecdule: 사용 x
- quit: Signal에 의한 종료
- reportState: State 보고
- join: ClusterManager에 등록
- keepAlive: Health check

## NodeManager

RpcServer를 생성하면서, 자신의 child process인 WorkerNode를 생성(spawn)한다. 또한 child process로 부터 주기적으로 health check를 받는다. 이때 WorkerNode가 WorkerAgent로 보내는 message는 node.js의 process에서 제공하는 api를 이용한다. WokerAgent는 RPC로써 getNode() / recycleNode() / queryNode()를 노출하며, conference agent가 WorkerNode간의 연결과 task를 관리하기 위해 호출한다.

- getNode(): WorkerNode를 생성 / 제공하며 roomId / sessionId와 WorkerNode를 매핑하여 관리
- recycleNode(): sessionId에 해당하는 WorkerNode의 동작을 중지, process kill, WorkerNode map에서 삭제
- queryNode(): sessionId에 해당하는 WorkerNode 반

## WorkerNode

WorkerNode는 ConfenceAgent가 pipeline을 구성하여 task를 할당하기 위해, WorkerAgent에 요청하여 생성된 process이다. 서로 간의 internal connection과 task가 session이라는 단위로 관리된다.  이러한 과정은 RpcClient 및 RpcServer로 동작하며 ConferenceAgent에 의해 컨트롤된다. 

하지만 process의 할당 / 해제는 WorkerAgent에 의해 관리 받는다.  WorkerAgent 에는 RpcMonitorTarget이라는 instance를 생성하고,  WorkerNode에는 RpcMonitor 라는 instance를 생성하여 "fault" message를 전달한다. 그리고 WorkerNode에서  fault message를 수신하게 되면 process를 종료한다.

WorkerNode는 purpose에 따라서 종류가 갈리게 된다. purpose에는 성격에 따라 다음과 같이 분류된다.

- Streaming gateway: QuicAgent, SipAgent, WebrtcAgent, StreamingAgent
- Media processing: VideoAgent, AudioAgent
- Recording: RecordingAgent

WorkerAgent는 모두 같은 소스코드로 동작하지만 purpose에 따라 다른 WorkerNode를 실행하게 되면서 다른 역할을 갖는다. 본 문서에서 정리 ClusterManager / WorkerAgent / WorkerNode / ConferenceAgent 사이의 관계는 Fig. 0을 통해서 표현했다.

![Relationship between Instances_](/assets/img/post/owt/worker_agent/worker_instances.png)
_Fig. 0: Relationship between Instances_

# References
- [0] [https://github.com/open-webrtc-toolkit/owt-server](https://github.com/open-webrtc-toolkit/owt-server)