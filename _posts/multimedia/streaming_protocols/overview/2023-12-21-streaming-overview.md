---
title: Overview (Streaming Protocols)
author: YonghyunCho
date: 2019-12-20 14:10:00 +0800
categories: [Multimedia, Streaming Protocols]
tags: [multimedia, streaming]
render_with_liquid: false
---

## Progressive download

progressive download는 server로 부터 HTTP를 통해 다운로드 받은 후, client에서 재생하는 가장 원시적인 streaming 방법이다. server에는 GET 요청에따라  파일을 내주는정도의 기능만 갖고 있으면 손쉽게 제공할 수 있다. 파일의 특정 지점을 재생하기 위해서는 그 지점까지의 모든 파일을 다운로드 받아야하며, 재생을 시작한 시점에서 파일의 끝까지 다운로드를 시도한다. 또한 모든 영상이 하나의 파일에 저장되기 때문에 Live streaming을 구현할 수 있는 방법은 아니다.

## Pseudo streaming

Progressive download를 하게되면서 server의 network bandwidth는 커졌고, 이를 보완하고자하는 기술이 등장한다. Client에서 파일의 헤더정보를 들고있고, 이를 파싱해서 필요할때마다 서버에 일부를 요청하는 방식이다. 또한 seek을 수행할때도 파일의 특정 지점부터 요청을 하기 때문에 progressive download에 비해 server/client 양쪽에 부하를 줄여준다. 

Pseudo streaming을 위해서는 client에서는 header 정보를 파싱하여 파일내 패킷의 위치를 조회한 후 파일의 일부를 요청하는 기능이 필요하다. 이런 구현을 위해서는 packet의 위치 정보를 헤더에 갖고있는 컨테이너가 필요하며, mp4/mkv와 같은 컨테이너가 이에 해당한다. 파일의 일부를 요청하는 기능은 HTTP 1.1부터 Range 헤더를 통해서 표준화 되어있다[0]. 

## RTMP (Real-Time Messaging Protocol)

RTMP는 Adobe의 Flash player에서 재생을 하기위해 개발한 real-time protocol이다. Multimedia data와 control command를 message화하여 전송한다. 1935번 포트를 기본포트로 사용하며, TCP 위에서 동작하지만 HTTP를 이용한 RTMPT, HTTPS를 이용한 RTMPS등의 확장한 버전도 있다.[1] 
Adobe에서 개발한 만큼 flv와 구조가 굉장히 닮아있다. 이름에서 보이는 만큼 real-time을 지향하며, 따라서 live streaming을 구현하기 적합하다. 하지만 브라우저에서 재생하기 위해서는 별도의 plug-in이 필요하다. plug-in을 브라우저에서 더이상 제공하지 않기 때문에 재생을 위해서는 사용이 어렵지만, server가 데이터를 수신하는 목적으로는 아직 유효하다. 

## HLS (HTTP Live Streaming)

Apple에서 개발한 adaptive HTTP streaming으로, 긴 재생시간을 갖는 파일을 짧은 재생시간을 갖는 여러개의 segment (file)로 나누어서 스트리밍하는 기술. RTMP과 비교하면 latency가 길어서 conferencing과 같이 interaction이 필요한 경우에는 사용하기 어렵지만, 중계와 같은 live streaming을 하기에 적합하다. 또한 HTTP를 통해서 동작하기 때문에 browser 환경에서의 제약이 없다. 

HLS는 미디어 데이터를 담고있는 segment 파일과, segment를 관리하는 playlist 파일로 나누어져 동작한다. playlist 파일의 경우에는 m3u8확장자를 사용하며, Fig. 0과 같이 tag와 데이터가 연속해서 나열되어 있는 형태이다. Playlist 파일에 사용되는 tag는 [2]문서에 명시되어있다. 미디어 데이터가 담겨있는 segment파일로 초기에는 segment로 ts 컨테이너만을 지원했지만, 2016년는 fmp4 (fragmented mp4)도 지원하기 시작했다. 

HLS 형태의 streaming이 가능해진 이유는 HTML5의 MSE (Media Source Extension)가 등장했기 때문이. MSE는 브라우저의 video tag를 통해서 미디어를 재생할때, source의 추가 동작을 구현할 수 있도록 확장시켜주는 표준이다.

![Playlist File for HLS](/assets/img/post/multimedia_protocol/streaming_overview/hls-playlist.png)
_Fig. 0: Playlist File for HLS_

## MPEG-DASH (MPEG Dynamic Adaptive Streaming over HTTP)

DASH또한 adaptive HTTP streaming 기술 중 하나로, HLS와 같이 segment와 Fig. 1과 같은 segment를 관리할 playlist (manifest)를 둔다. 그러나 DASH는 playlist로써 xml 형태의 manifest 파일을 두고, segment로는 webm과 fmp4 컨테이너를 사용한다. 하지만 초창기 HLS에서 ts를 segment로 사용했기 때문에 다양한 codec의 파일을 제공할 수 없었다. 따라서 DASH의 manifest파일은 mpd 확장자를 가진다. HLS의 playlist와 비교했을 때 확장하기 쉬다. 또한 재생시간이 증가함에따라 segment가 계속 늘어난다 해도, manifest파일은 무한히 길어지지 않는다.

![Manifest File for DASH](/assets/img/post/multimedia_protocol/streaming_overview/dash-manifest.png)
_Fig. 0: Manifest File for DASH_

## CMAF (Common Media Application Format)

Adaptive HTTP Streaming이 늘어남에 따라, 스트리밍을 위한 application들은 각각의 Adaptive HTTP Streaming의 parser와 DRM encryption/description을 수행하는 구현체를 각각 갖고 있어야 한다. 또한 storage에 저장되어 있는 미디어 또한 각각의 포맷에 맞게 converting해야 하니 효율성 또한 떨어진다. Microsoft와 Apple이 MPEG에 표준을 제안함으로써 복잡한 표준을 정했다. CMAF의 주요한 표준에는 다음의 세 가지 내용이 있다.

- fmp4로 컨테이너 단일화
- MPEG-CENC (ISO/IEC 23001-7)로 DRM 단일화
- ULL (Ultra Low Latecny)

# References
- [0] [https://tools.ietf.org/html/rfc7233](https://tools.ietf.org/html/rfc7233)
- [1] [https://en.wikipedia.org/wiki/Real-Time_Messaging_Protocol](https://en.wikipedia.org/wiki/Real-Time_Messaging_Protocol)
- [2] [https://tools.ietf.org/html/rfc8216](https://tools.ietf.org/html/rfc8216)