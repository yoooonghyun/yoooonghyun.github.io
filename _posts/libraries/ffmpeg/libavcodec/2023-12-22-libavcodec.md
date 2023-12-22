---
title: libavcodec
author: YonghyunCho
date: 2019-12-20 14:10:00 +0800
categories: [Libraries, FFmpeg]
tags: [ffmpeg]
render_with_liquid: false
---

FFmpeg에서 Decoding와 Encoding을 지원하는 codec 구현이 있다. 외부 libopus / libvpx / libx264 와 같은 외부 lib을 이용하는 경우도 있으며, 자체 구현된 codec도 존재한다. Codec과 관련된 코드가 모두 포함되어 있기 때문에 FFmpeg source중 가장 방대하다. 

## Struct

### AVCodec

FFmpeg에서 제공하는 codec들을 추상화한 struct. Codec 구현체는 각각 AVCodec의 function pointer와 member들을 채운 구현체를 노출한다. AVCodec instance를 global list로 생성하며, 해당 list를 통해서 codec을 조회할 수 있다. Codec custom하게 채워진 init, encode2, decode, send_frame, receive_packet, receive_frame, close 등의 함수가 공통적으로 제공하는 함수 안에서 호출됨에 따라, 각각의 codec별로 다른 동작을 수행한다.

### AVCodecContext

Encoding / Decoding에 대한 context를 갖는 struct로, encoding / decoding관련 함수가 호출될 때 parameter로 넣는 struct이다. 특히 encoding할 때 frame의 pixel format, quality, quantization에 대한 옵션등을 추가할 수 있다. 

AVCodecContext의 멤버 중 주목할만한 내용은 get_buffer2이다. Multimedia pipeline을 구성하여 decoding module에 FFmpeg을 사용하였다고 해보자. 이때, buffer pool을 두어 미 할당한 memory 공간에 decoded 데이터를 담아야 하는 경우가 발생할 수 있다. 또한 FFmpeg과 다른 buffer를 사용하며 reference 문제가 발생할 수 있다. 이 같은 문제를 해결하기 위해서 AVCodecContext::get_buffer2에 custom function을 채워 줌으로써 decoded data가 채워지는 memory 공간에 대한 제어가 가능하다. get_buffer2는 기본적으로 decode.c 파일에서 정의된 avcodec_default_get_buffer2() 함수로 채워져 있다. 

![Structs Provided by libavcodec](/assets/img/post/ffmpeg/libavcodec/av_codec_struct.png)
_Fig. 0: Structs Provided by libavcodec, (a): AVCodec, (b): AVCodecContext_

### AVFrame

AVFrame은 raw media data를 추상화한 struct이다. [Media sampling](/posts/media-sampling)문서에서 기술한 planar / semi-planar / packed video data와 planar / interleaved audio data를 모두 추상화 할 수 있다. AVFrame이 담고 있는 정보는 크게 frame data와 data에 대한 context로 나뉜다. 주요 member에는 data의 주소, data size, pixel / sample format, PTS (Presentation Time Stamp), DTS (Decoding Time Stamp)가 있다.  

data에는 최대 8개의 plane을 담을 수 있으며, 각각 plane의 주소를 가진다. size 영역에는 video의 경우 line size를 audio의 경우 plane의 data size를 갖는다. 각각의 data 사이즈는 아래 그림과 같이 계산된다.

![Data Abstraction in AVFrame](/assets/img/post/ffmpeg/libavcodec/frame_data.png)
_Fig. 1: Data Abstraction in AVFrame, (a): Video Data, (b): Interleaved Audio data, (c): Planar Audio Data_

Video pixel format와 audio sample format은 각각 enum type으로  AVPixelFormat과 AVSampleFormat에 명시되어 있다. PTS와 DTS는 각각 재생되어야 할 시간과  decoding을 수행해야 할 시간의 time stamp를 의미한다. FFmpeg의 시간은 time base라는 단위를 정의하여 표기된다. Time base는 유리수 (Rational number)로 정의되며, 시간 단위를 몇 초로 나타낼 지를 정한다. 즉, Presentation time = [PTS] x [time base] s로 계산할 수 있다.

AVFrame의 데이터 영역은 AVBufferRef * buf 멤버를 두고 atomic reference count를 관리를 하고 있다. 따라서 frame 데이터를 다른 frame으로 copy할 때는 av_frame_ref(), av_frame_move_ref()함수를 사용해야 한다.

### AVPacket

AVPacket은 압축된 데이터인 packet을 추상화한 struct이다. Packet은 데이터 내부에 decoding을 위한 table을 포함하고 있기 때문에 data의 context를 가지고 있는 AVFrame보다 struct가 포함하는 내용이 적다. 또한 encoding된 packet은 하나의 데이터 뭉치로 떨어지기 때문에 간단하게 data를 표현할 수 있다.

![Structs Used by libavcodec](/assets/img/post/ffmpeg/libavcodec/buffer_struct.png)
_Fig. 2: Structs Used by libavcodec, (a): AVPacket, (b): AVFrame_


## Function

### Find codecs

FFmpeg에서는 지원 가능한 codec 구현체에 대핸 codec을 추상화한 AVCodec을 list로 들고 있다. (allcodecs.c 참) 그리고 Codec ID 혹은 codec name을 통해서 encoder와 decoder를 조회할 수 있는 다음과 같은 함수를 제공한다.

``` C
AVCodec *avcodec_find_decoder(enum AVCodecID id)

AVCodec *avcodec_find_decoder_by_name(const char * name)

AVCodec *avcodec_find_encoder(enum AVCodecID id)

AVCodec *avcodec_find_encoder_by_name(const char * name)
```

### Codec initialize

Encoding / Decoding을 하기 위해 libavcodec의 주요 함수들은 AVCodecContext를 주요 인자로 받는다. AVCodecContext에는 AVCodec과 프로세싱을 위한 context가 추상화 되어있다. FFmpeg 4.1을 기준으로 AVCodecContext의 초기화는 다음의 함수를 통해 이루어진다.

``` C
int avcodec_open2(AVCodecContext *avctx, const AVCodec *codec, AVDictionary **options)
```

parameter로 들어가는 options를 통해 encoding / decoding option들을 추가할 수 있다. 추가 가능한 리스트와 값들은 options_table.h에서 조회할 수 있다.

Child thread를 사용한 encoding / decoding을 하기 위해서 필요한 thread도 이 단계에서 생성된다.  Encoding thread를 생성하기 위해서는 CONFIG_FRAME_THREAD_ENCODER flag가 올라가 있어야 하며, 해당 flag는 frame_thread_encoder.o object가 빌드됨에 따라 올라간다. 또한 Decoding HAVE_THREADS flags에 의해 활성화 되며, pthread.o / pthread_slice.o / pthread_frame.o object들의 빌드 결과에 따라 올라간다.

### Decoding

FFmpeg의 decoding과정은 AVPacket을 decoder에 밀어 넣는 과정과, decoder로 부터 AVFrame을 받는 두 가지 과정으로 나뉘며 각각의 경우에 다음의 함수를 이용한다.

``` C
int avcodec_send_packet(AVCodecContext *avctx, const AVPacket *avpkt)
int avcodec_receive_frame(AVCodecContext *avctx, AVFrame *frame)
```

avcodec_send_packet() 함수는 decoder 내부로 decoding할 packet을 밀어 넣는 과정으로, Fig. 3 (a)는 avcodec_send_packet()  함수의 내부 동작을 묘사한 그림이다. User로부터 packet을 받았을 때, AVCodecContext 내부에 decoding된 buffered frame을 cached하고 있지 않다면, 하나의 frame을 cache하기 위해서 Fig. 3 (c)에서 묘사하는 decode_receive_frame_internal() 함수를 호출하여 packet을 바로 decoding한다.

avcodec_receive_frame() 함수는 decoder로부터 frame을 받아내는 함수로 Fig. 3 (b)를 통해 묘사되고 있. buffered frame이 있다면 후리처리하여 바로 반환하지만, buffered frame이 없는 경우에는 decode_receive_frame_internal()을 호출하여 decoding을 수행한다.
Fig.3 (c)에서 묘사되는 decode_receive_frame_internal() 함수는 실제로 codec 구현체의 decode 로직을 호출하는 부분으로 다음의 세 가지 형태로 동작한다.

- Codec 구현체의 receive_frame()을 호출
- Codec 구현체의 decode()를 호출
- Child thread에서 Codec 구현체의 decode()를 호출

FFmpeg decoder 구현체는 receive_frame() / decode() 중 하나의 함수를 구현해야 한다. receive_frame()을 구현하는 경우에는 자유도 있는 동작을 할 수 있지만, timestamp를 채우거나 decoder를 위한 thread를 할당하는 등의 작업을 코덱 구현체 내부에서 해야한다.

![Decoding Progress in FFmpeg](/assets/img/post/ffmpeg/libavcodec/decoding_uml.png)
_Fig. 3: Decoding Progress in FFmpeg, (a) Send Packet, (b) Receive Frame, (c) Internal Receive Frame_

### Encoding

FFmpeg의 encoding과정은 decoding과정과 대칭적으로 AVFrame을 encoder에 밀어 넣는 과정과, encoder로 부터 AVPacket을 받는 두 가지 과정으로 나뉘며 각각의 경우에 다음 함수를 이용한다.

``` C
int avcodec_send_frame(AVCodecContext *avctx, const AVFrame *frame)
int avcodec_receive_packet(AVCodecContext *avctx, AVPacket *avpkt)
```

Encoder의 동작은 decoder의 동작과 정확하게 대칭적이다. 각각의 과정은 Fig. 4에 flow chart를 이용하 묘사해 놓았다. 


![Encoding Progress in FFmpeg](/assets/img/post/ffmpeg/libavcodec/encoding_uml.png)
_Fig. 4: Encoding Progress in FFmpeg, (a) Send Frame, (b) Receive Packet, (c) Internal Receive Packet_


### Codec deinitialize

Codec을 deinitialize하는 과정으로 initialize시에 생성한 context를 해제한다. Memory leak을 피하기 위해서 deinitialize를 수행하기 전에는 codec 내부의 buffer에 대한 flush를 수행해야 한다. Deinitialize / Flush를 수행하 함수는 다음과 같다.

``` C
int avcodec_close(AVCodecContext *avctx)
void avcodec_flush_buffers(AVCodecContext *avctx)
```

# References

- [0] [http://ffmpeg.org/doxygen/4.1](http://ffmpeg.org/doxygen/4.1)
- [1] [https://github.com/FFmpeg/FFmpeg/tree/release/4.1](https://github.com/FFmpeg/FFmpeg/tree/release/4.1)