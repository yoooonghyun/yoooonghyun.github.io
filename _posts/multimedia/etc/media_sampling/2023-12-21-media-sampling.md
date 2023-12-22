---
title: Media Sampling
author: YonghyunCho
date: 2019-12-21 14:10:00 +0800
categories: [Multimedia, ETC]
tags: [multimedia]
render_with_liquid: false
---

Multimedia에서는 연속적인 신호인 그림이나 소리 정보를 디지털 정보를 변환하기 위해 sampling 과정을 거친다. Sampling 과정을 거치면서 audio/image 는 time/distance 축에 대해 **discontinuous** (불연속) 한 특성을 가진다. 또한, 소리/빛의 세기를 나타내는 수치가 특정 자료형을 통해서 표현되면서 그 수치는 **quantization** (양자화)된다.

## Audio Sampling

Sound는 가청주파수 (20 - 20000 Hz) 범위 공기를 매질로 하는 파동이다. 파동은 시간 축에 대해 continuous하며, 그 진폭 (amplitude) 또한 연속적인 값을 갖는다. 하지만 이를 디지털 장비에서 다루기 위해서는 불연속적이며 양자화된 데이터로 변환해 한다. Fig. 0은 시간 t에 대해 연속적인 신호 S (t ) 의 sampling을 보여준다. S (t )를 일정한 주기 T 로 나누게 되면 각 지점에서 순간적인 대표 값을 가지게 된다. 이 값들은 특정 자료형에 담겨 디지털 데이터가 되는데, 이 데이터 각각을 **sample**이라고 부른다. **sample**들을 analog 신호로부터 추출하는 과정을 sampling이라 한다. Sample들간의 일정한 간격 T 를 sampling period라 하며, T 의 역수를 sampling frequency라 한다. 특정 주파수 대역 f 를 관측하기 위해서는 sampling frequency가 f 의 2배 이상이어야 한다 (Nyquist sampling theorem).

![Sampling of Audio Signal](/assets/img/post/multimedia_etc/media_sampling/audio-sampling.png)
_Fig. 0: Sampling of Audio Signal [0]_

## Audio Sample Format

각각의 audio sample은 audio 신호의 순간적인 amplitude를 의미한다. 이 값은 주로 integer 혹은 floating point arithmetic을 이용하여 표현다. Sample을 표현하 자료형의 범위가 클수록 신호의 amplitude를  세분화해서 표현할 수 있으며, 따라서 더 좋은 음질을 구현할 수 있다. 각 자료형 범위는 Fig. 1에 정리해 두었다.

![Ranges of Each Data Type](/assets/img/post/multimedia_etc/media_sampling/range-of-number.png)
_Fig. 1: Ranges of Each Data Type_

Sampling된 audio data는 일반적으로 각 sample을 단독적으로 연산하 않고, buffer를 이루어 한번에 여러 개의 sample이 연산한다. 이런 한 번에 처리되는 audio sample buffer 단위를 audio  frame이라고 부르면, 각 audio frame은 목적에 따라 interleave type과 planar type (non-interleaved type) 으로 나뉜. Fig. 2는 stereo에서 각각의 타입을 묘사한 것이다. 

![Types of Audio Frame](/assets/img/post/multimedia_etc/media_sampling/audio-frame.png)
_Fig. 2: Types of Audio Frame, (a): Interleaved Type, (b): Planar Type_

- (a): Interleaved type은 각 채널의 sample이 번갈아가면서 (interleaving) 하나의 버퍼에 담겨져 있다. 일반적인 audio driver가 interleaved type 데이터를 주고받으며, digital-to-analog 변환을 하기 위하거나 analog-to-digital 변환을 하기 위한 데이터 형태이다.  
- (b): Planar type은 각 채널이 독립적인 buffer에 담겨있는 형태이다. Codec에서는 frequency 특성을 이용하기 위해 DCT와 같은 연산을 수행하는데, 각 채널 별로 정보를 연산하기 위해 사용한다. 즉, audio codec의 input/output을 위한 타입이다.

## Video Sampling

Video는 각각의 image가 연속적으로 보여지는 것로, 각각의 image가 sampling 되어 디지털 정보로 변환된다. Audio와 마찬가지로  디지털 기기에서 image를 처리하기 위해는 analog 데이터를 연속적이고 양자화 되어있는 데이터로 변환해야 한다.  Fig. 3은 카메라 센서에 상이 맺히는 것을 표현한 그림이다. 사물/배경에 의해 반사된 빛은 카메라 렌즈를 통해 굴절되어 격자화 되어있는 센서에 상이 맺힌다. 센서의 각 격자에는 들어온 빛의 세기 값을 측정하여 디지털 신호를 만들어낸다. 그런 각자의 격자는 사람의 원추세포와 같이 Red/Green/Blue (RGB) 센서로 나누어져 있으며, 이 RGB 데이터를 이용하여 한 번의 색 표현을 한다. 한 번의 색 표현을 하는 단위를 **pixel**이라 하며,  set of pixel은 한 장의 image를 구성한다. 이와 같이 sampling 된 한 장의 image를 video frame이라고 부르면, **video frame**의 연속적으로 변화를 통해 video는 움직이는 영상을 보여준다.

![Concept of Video Sampling](/assets/img/post/multimedia_etc/media_sampling/video-sampling.png)
_Fig. 3: Concept of Video Sampling_

## Video Pixel Format

Multimedia에서 video 데이터는 입/출력을 위한 RGB format이 있고, 압축을 위한 YUV format로 나뉜다. RGB는 디바이스에서의 입/출력 및 렌더링을 위한 형태로, Red / Green / Blue로 구성된 colorspace를 이용해서 색을 표현하는 것이다. 그리고 YUV는 명도 값인 Y 와 색차정보인 U/V 값을 이용하여 색 표현을 한다. BT.709에서 명시된 RGB와 YUV사이의 변환식은 Fig. 4에서  보여지는 것과 같다. [1]

![Relation between RGB and YUV](/assets/img/post/multimedia_etc/media_sampling/rgb-yub-conversion-equation.png)
_Fig. 4: Relation between RGB and YUV [1]_

### RGB Pixel Format

RGB format으로 색표현을 하 video frame은 거의 모든 경우에 packed 데이터를 다룬다. Packed란 audio sample format에서 interleaved와 같이, 각각의 색 요소가 번갈아 가면서 하나의 plane에 담겨있는 형태이다.  RGB를 표기할 때는 Red / Green / Blue 각각의 요소가 얼마만큼의 bit로 표현되는 지, 배열 순서는 어떻게 되는 지에 따라 표기한다. 예를 들면,

- RGB565 (RGB16): Red / Green / Blue 순으로 데이터가 정렬되어 있으며, 각각 5 / 6 / 5 bit depth
- BGR888 (BGR24): Blue / Green / Red 순으로 데이터가 정렬되어 있으며, 각각 8 / 8 / 8 bit depth

RGB pixel format에는 blending을 위해서 투명도 정보인 A (alpha) 요소를 추가하거나, octet align를 위해 padding bit를 의미하는 X요소를 추가하기도 한다. Alpha값이 포함된 경우는 위에서 설명한 규칙으로 RGBA8888 (RGBA32)와 같은 형태로 표기한다. 그리고 비어있는 bit가 포함된 경우는 XRGB1555 (XRGB15)와 같이 표기한다 (15로 표기하지만 실제 16 bit를 점유). 더 많은 RGB Pixel format이 궁금하면, Reference [2]에 linux의 v4l2에서 사용하는 다양한 RGB pixel format에 대한 표기, fourcc code, align이 정리되어 있다.

### YUV Pixel Format

사람의 눈에는 명암을 구분하는 간상세포가 색을 구분하는 원추세포보다 많기 때문에, 색 변화에 비해 밝기 변화에 더 민감하다. 이 사실을 이용하여 밝기 정보 (Y)에 비해 색차 정보 (Cr/Cb, U/V)를 줄여서 영상을 encoding하는데, 이를 chroma subsampling이라고 한다. Chroma subsampling된 YUV 데이터 Y값은 모든 sample (pixel)에 대해서 표현하지만, U/V 값은 주위의 sample을 합친 단위인 sub sample에 대한 하나의 값만 표현한다. 이때 YUV format은는 어떤 subsampling을 거치는  지에 따라 이름이 정해진다. 

![Principle of Chroma Subsampling](/assets/img/post/multimedia_etc/media_sampling/chroma-subsampling.png)
_Fig. 5: Principle of Chroma Subsampling [3]_

Subsample과정을 표현한 Fig. 5를 보면 4 x 2의 sample들에 대해,
- J: 각 row에 존재하는 Y sample의 갯수
- a: 첫 번째 row의 chroma sample의 갯수
- b: 첫 번째 row와 두 번째 row에서 chroma sample이 변경 갯수
로 J, a, b 값이 정해지며, 4:2:0 subsampling을 거친 YUV format은 YUV420과 같이 표기한다.

RGB format이 대부분의 경우 packed로 저장되는 반면에, YUV format은 packed / semi-planar / planar의 형태로 다양하게 저된다. YUV422를 예시로 들자면 YUV422 format은 Fig. 6처럼 세 종류의 형태로 나타낼 수 있다.

![Different Type of YUV422 format](/assets/img/post/multimedia_etc/media_sampling/yuv-example.png)
_Fig. 6: Different Type of YUV422 format , (a): Packed, (b): Semi-planar, (c): Planar_

- (a) YUYV: Packed 형태로 Y/U/V가 모두 하나의 plane에 interleaving되어 있다.
- (b) NV16: Semi-planar 형태로 Y는 독립된 plane에 저장되고, U/V는 함께 interleaving되어 있다.
- (c) I422: Planar 형태로 Y/U/V가 각각 독립된 plane에 있다.

# References
- [0] [https://en.wikipedia.org/wiki/Sampling_(signal_processing)](https://en.wikipedia.org/wiki/Sampling_(signal_processing))
- [1] [https://www.itu.int/dms_pubrec/itu-r/rec/bt/R-REC-BT.709-6-201506-I!!PDF-E.pdf](https://www.itu.int/dms_pubrec/itu-r/rec/bt/R-REC-BT.709-6-201506-I!!PDF-E.pdf)
- [2] [https://www.kernel.org/doc/html/v4.9/media/uapi/v4l/pixfmt-packed-rgb.html](https://www.kernel.org/doc/html/v4.9/media/uapi/v4l/pixfmt-packed-rgb.html)
- [3] [https://en.wikipedia.org/wiki/Chroma_subsampling](https://en.wikipedia.org/wiki/Chroma_subsampling)