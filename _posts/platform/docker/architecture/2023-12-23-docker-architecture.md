---
title: Architecture (Docker)
author: YonghyunCho
date: 2019-12-20 14:10:00 +0800
categories: [Platforms, Docker]
tags: [docker]
render_with_liquid: false
---

## Docker Engine

Docker Engine은 Fig. 0에 묘사된 Container를 호스팅하는 소프트웨어이며, 설치된 Docker 자체라고 볼 수 있다. Docker Engine은 server-client구조를 띠 다음의 요소를 갖고 동작한다. [0]

- Docker Daemon (dockerd)
- Docker Daemon API
- Docker Client (docker)

![Docker Engine](/assets/img/post/docker/architecture/docker-engine.webp)
_Fig. 0: Docker Engine [1]_

Docker를 구성하는 요소에는 daemon / client / registry / obeject이 있다. Docker Client와 Docker Daemon는 서로 UNIX socket / network interface위에서 동작하는 REST API를 이용하여 통신한다. 일반적으로 Daemon과 Client는 서로 같은 시스템에서 동작하지만, Client와 remote Daemon을 연결하여 사용 가능하다. 

![Docker Architecture](/assets/img/post/docker/architecture/docker-architecture.webp)
_Fig. 1: Docker Architecture [2]_

## Daemon (dockerd)

Docker를 설치하면 dockerd(Docker Daemon)가 설치된다. Docker Daemon은 Docker API를 받아 image / container / network / volume과 같은 object들을 관리하는 역할을 한다. dockerd는 Reference [3]의 가이드에 제공되는 options과 함께 실행할 수 있다.

## Client

### docker

Docker Client는 사용자가 Docker와 통신하는 가장 기본적인 방법이다. docker 명령어를 통해 실행되며, Docker API를 Docker Daemon에 전송함으로써 동작한다. 제공하는 명령어와 옵션은 Reference [3]에 기술되어 있다.

### docker-compose

여러 개의 Docker Container로 이루어진 application을 개발할 때 사용 가능한 tool이다. YAML file에 application을 구성하는 service의 정보를 기술한 후, docker-compose CLI를 이용하여 제어한다. 관련 명령어는 Reference [2]에 기술되어 있다.

## Registry

Docker Registry는 Docker Image를 저장하는 역할을 한다. Docker Hub라는 public registry가 존재하며, 누구나 이용이 가능하며, private registry를 만드는 것 또한 가능하다. 사용자는 docker pull / run / push와 같은 명령어를 통해 Docker Registry에 접근이 가능하다.

## Object

### Image
Docker Image는 Docker Container를 생성하기 위한 template이다. Docker Image에는 application, dependency, file system, evironment 등의 정보가 저장되어 있다. Image는 Dockerfile을 작성함으로써 생성할 수 있다. 이때, Image는 다른 Image를 기반으로 만들어질 수 있으며, 변화한 부분에 대한 layer만 수정된다. 이러한 동작 방식은 Docker를 더욱 간편하고 빠르게 만들어준다.

### Container
Docker Container는 Docker Image를 실행할 수 있도록 만든 instance이다. Docker API / CLI를 이용하여 제어할 수 있으며, network / storage와 언결할 수 잇다. 또한 실행하고 있는 Docker Container를 기반으로 Image를 생성하여 현재 상태를 저장할 수 있다.
Docker Container는 host system / 다른 container와 마치 독립되어 동작하지만, network와 volume을 서로 공유 / 연결할 수 있다.

### Network

Docker는 Container를 서로 연결시키거나 non-Docker환경과도 연결할 수 있다. Docker networking subsystem은 Docker에서 제공하는 driver를 이용하여 변경될 수 있으며, 다음과 같은 형태 중 선택하여 기능을 사용할 수 있다.

- bridge: 기본 network driver. Docker Container를 외부와 통신시키기 위해 일반적으로 사용.
- host: Host system의 network를 직접 사용하여, Container와 host간의 network isolation을 하지 않음.
- overlay: 서로 다른 Docker host를 갖는 container간의 통신을 지원. Docker swarm service를 서로 통신할 수 있게 해준다.
- macvlan: Container가 MAC 주소에 접근 가능하게 해줌. 오래된 application을 사용하는 상황에서, physical network에 직접 접근해야 하는 경우에 좋은 선택지가 될 수 있다.
- none: Networking을 비활성화 시킴. Custom network driver와 연결하는 경우 이용.
- Network plugins: Docker Hub 혹은 thrid-party vendor에서 만든 custom network plugin을 설치하여 사용 가능.

## File Format

### Dockerfile

Docker Image를 빌드하기 위해 필요한 파일이다. 해당 파일을 포맷에 맞게 작성한 후, docker build 명령어를 통해 Docker Image 빌드가 가능하다. **Reference [3]**에 Dockerfile에서 지원하는 문법이 기술되어있다.

### docker-compose.yaml

docker-compose를 실행하기 위한 내용을 기술하는 파일이다. Runtime 환경의 service / network / volume에 대한 기술을 할 수 있다. 버전에 따라 사용할 수 있는 문법이 나뉘어 있으며, Compose file reference라는 이름으로 Reference [3]에 포맷들이 명시되어 있다.

# References

- [0] [https://docs.docker.com/engine/](https://docs.docker.com/engine/)
- [1] [https://www.docker.com/products/container-runtime](https://www.docker.com/products/container-runtime)
- [2] [https://docs.docker.com/get-started/overview/](https://docs.docker.com/get-started/overview/)
- [3] [https://docs.docker.com/reference/](https://docs.docker.com/reference/)