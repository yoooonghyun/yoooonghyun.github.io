---
title: Pipeline (OWT)
author: YonghyunCho
date: 2019-12-20 14:10:00 +0800
categories: [Libraries, OWT]
tags: [owt]
render_with_liquid: false
---

## Pipeline

Javascript는 mulitmedia processing를 하기에는 성능적인 측면에서 적합하지 않다.  OWT에서는 Node js를 이용한 RPC통신과 resource 관리 수행할 뿐, 실제 mulitmedia processing은 C++ / C addon을 통해서 처리한다. Streaming gateway, InternalConnection, multimedia processor의 동작 구현은 source/agent/addons 혹은 source/core/owt_base에 위치한다.

OWT의 pipeline을 구성하는 PipelineComponent는 Fig. 0에서 표현한 FrameSource / FrameDestination 이라는 interface를 이용하여 구현한다. 각각의 PipelineComponent은 Fig. 0 (a) / (b) / (c)와 같이 FrameSource의 특성을 띄거나, Frame Destination의 특성을 띄거나, 두 가지 특성 모두 띌 수 있다. 이때 FrameSource에서 FrameDestination으로 header정보를 의미하는 MetaData와 추상화된 media data인 Frame이 전달된다. 그리고 반대 방향으로는 bitrate를 설정하거나 key frame을 요청하는 feedback message를 전달한다.

![OWT Pipeline](/assets/img/post/owt/pipeline/owt_pipeline.png)
_Fig. 0: OWT Pipeline, (a): Source Component, (b): Destination Component, (c): Both, (d): Transport Data_

FrameSource와 FrameDestination은 pipeline의 구성과 데이터 전달을 위한 interface만 제공할 뿐, 이로 인한 구현의 제약이 크지 않다. InternalConnection을 포함한 PipelineComponent의 내부 동작은 component custom하게 다양한 library를 사용하여 구현되어 있다.

FrameSource와 FrameDestination사이의 연결을 통한 pipeline은 WorkerNode내부에 존재한다. 그리고 PipelineComponent의 종단부는 항상 I/O와 연결되어 있어, WorkerNode간의 데이터를 주고받거나, 외부의 client와 데이터를 교환한다. WebrtcWorkerNode / VideoWorkerNode / QuicWorkerNode로 구성된 server전체의 data의 흐름은 Fig. 1 과 같이 구성된다.

![Overall Data Flow](/assets/img/post/owt/pipeline/data_flow.png)
_Fig. 1: Example of Overall Data Flow in OWT Server_

## Node-gyp

NodeJS는 CommonJS 표준과 V8 engine을 기반으로 만들어진 JS runtime이다. V8 engine은 JS 코드를 기계어로 바꾸어 실행시킨다. V8는 C++로 작성되어 있으며, C++로 작성한 모듈도 compile한 후 NodeJS에서 require하여 사용할 수 있도록 제공한다. 이렇게 C++로 짜여진 module을 C++ addon / Native Addon Module이라한다. 이때, compile을 node-gyp라는 툴을 이용하며 compile 결과 .node 확장자 형태의 파일을 생성한다. NodeJS 코드에서는 다른 JS파일과 같이 이 .node 파일을 require하여 사용 가능하다. Reference [2]에는 이에 대한 가이드가 있으니 참고하여 C++ addon을 구성하면 된다.

node-gyp를 이용한 과정을 StreamingAgent에서 사용하는 AVStream을 빌드하는 과정을 통해 살펴보자. node-gyp build를 하기 위해서는 Fig. 2과 같은 binding.gyp을 작성해야 한다. binding.gyp에는 target name, source code, 링크할 lib, C++ compile 옵션 등을 기술한다.

![binding.gyp](/assets/img/post/owt/pipeline/gyp.png)
_Fig. 2: binding.gyp [0]_

Fig. 3는 binding.gyp에서 기술되어있는 addon.cc 파일이다. addon.cc에서는 NODE_MODULE이라는 macro를 실행시키는데, 인자로 NODE_GYP_MODULE_NAME과 InitializerFunction을 입력해준다. Initializer Function은 Fig. 4에 보이는 함수와 같이, class name과 prototype을 구성하여 export하는 역할을 수행한다.

![addon.cc](/assets/img/post/owt/pipeline/addon.png)
_Fig. 3: addon.cc [0]_

![Initialize Function](/assets/img/post/owt/pipeline/initialize_function.png)
_Fig. 4: Initialize Function [0]_

Fig. 5 / Fig. 6에서는 Fig. 4에서 구성한 prototype의 생성자 및 method를 구현한다. 이때 인자는 V8 library에서 제공하는 FunctionCallbackInfo로 입력되기 때문, 예제에 보이는 것처럼 인자들을 parsing하여 사용한다.

![addon.cc](/assets/img/post/owt/pipeline/constructor.png)
_Fig. 5: Constructor [0]_

![Exported Methods](/assets/img/post/owt/pipeline/exported_method.png)
_Fig. 6: Exported Methods [0]_

compile 결과, target name에 해당하는 avstream.node로 결과물이 생성된다. StreamingWorkerNode에서는 이를 require한 후, initialize 단계에서 정의한 class name을 이용하여 사용한다.

![Usage in JS](/assets/img/post/owt/pipeline/usage_js.png)
_Fig. 7: Usage in JS [0]_

# References

- [0] [https://github.com/open-webrtc-toolkit/owt-server](https://github.com/open-webrtc-toolkit/owt-server)
- [1] [https://github.com/nodejs/node](https://github.com/nodejs/node)
- [2] [https://nodejs.org/api/addons.html](https://nodejs.org/api/addons.html)