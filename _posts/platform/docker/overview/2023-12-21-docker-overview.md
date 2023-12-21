---
title: Overview (Docker)
author: YonghyunCho
date: 2019-12-20 14:10:00 +0800
categories: [Platform, Docker]
tags: [platform, docker]
render_with_liquid: false
---

Docker는 container라는 형태로 software를 감쌈으로써 OS-level virtualization 제공하는 PaaS (Platform as a Service)이다. Container는 실행할 software, file, environment를 담아, 독립된 실행 환경을 제공한다. 또한 container끼리는 well-defined channel을 통해 서로 통신이 가능하다 [0]. 

패키징된 container는 docker가 설치된 Linux, Windows, MacOS의 운영체제에서 실행이 가능하다. 또한 on-premise, public / private cloud와 같은 다양한 환경에서 동작한다. Linux를 기준으로 docker는 자체적인 component인 libcontainer와 Linux kernel에서 지원하는 libvirt / LXC / systemd-nspawn을 통해 가상화를 지원한다. 

![Manifest File for DASH](/assets/img/post/streaming_overview/dash-manifest.png)
_Fig. 0: Docker & Hypervisor [1]_

Container와 VM(Virtual Machine)은 모두 resource의 독립과 할당에 대해 장점을 갖지만, Pic. 0에 묘사되 다른 동작 방식을 갖는다. [1]

- Container는 app layer를 추상화 하며, code와 dependency를 포함한다. 여러 Container는 같은 OS kernel과 기기를 공유한다. 따라서 Container는 VM에 비해 적은 공간을 필요로 하며, 더 많은 Container를 동작할 수 있다.
- Virtual Machine은 physical hardware를 추상화한다. Hypervisor는 하나의 기기위에 여러 VM을 동작하도록 허용한다. VM 안에는 OS가 각각 존재하기 때문에 많은 공간을 차지한다.

# References

- [0] [https://en.wikipedia.org/wiki/Docker_(software)](https://en.wikipedia.org/wiki/Docker_(software))
- [1] [https://www.docker.com/resources/what-container](https://www.docker.com/resources/what-container)