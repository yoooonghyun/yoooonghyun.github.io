---
title: Overview (FFmpeg)
date: 2019-12-20 14:10:00 +0800
categories: [Libraries, FFmpeg]
tags: [ffmpeg]
render_with_liquid: false
---

## Fmpeg Project

FFmpeg은 multimedia 전반적인 프로세싱 기능을 제공하는  C언어 기반의 library이다. encoding, decoding, muxing, demuxing과 같은 기능 뿐만아니라 video scalier (pixel converter), audio resampler 및 다양한 filter를 제공한다. 또한 ffmpeg/ffplay/ffprobe와 같은 binary도 제공하고있기 때문에, pipeline을 직접 구현하지 않고도 binary들을 실행시킴으로써 FFmpeg에서 제공하는 기능을 사용할 수 있다. **Reference [0]**, **Reference [1]**에 documentation이 되어 있으며, 많은 사람들이 사용하기 때문에 문제가 발생해도 쉽게 해결할 수 있다.

![FFmpeg Sources](/assets/img/post/ffmpeg/overview/ffmpeg_sources.png)
_Fig. 0: FFmpeg Sources_

**Reference [2]**에서 FFmpeg의 src를 받아보면 Fig. 0과 같다. FFmpeg lib들은 libavformat, libavcodec, libavdevice, libavutil, libavresample, libavswscale 디렉토리에 나뉘어 구현이 되어있으며, 빌드 시 각각의 lib으로 설치된다. tools에는 FFmpeg에서 제공하는 binary들을 포함하고 있다. 각각의 lib은 다음의 내용을 포함하고 있다.
- libavformat: File / Network io, container parser (muxing / demuxing)
- libavcodec: Codec (encoder / decoder)
- libavresample: Audio sample converting, audio resampling (up / down sampling)
- libavswscale: Video pixel converting, video resizing
- libavutil: Pixel format 변환, time scale 변환과 같은 함수
- libavdevice: Media device 입/출력을 위한 generic framework 


## Install

FFmpeg의 source를 다운로드 받아 build를 할 때는, FFmpeg 디렉토리에서 다음의 command를 순차적으로 입력하면 된다.

``` bash
$ ./configure --prefix=[PREPIX_TO_INSTALL] --extra-cflags=[C_FLAGS] --extra-ldflags=[LD_FLAGS] --bindir=[BIN_DIR_TO_INSTALL] --enable-[LIB_1] --enable-[LIB_2] ... --enable-[LIB_N]
$ make
$ make install
```

위의 첫 번째 단계는, build configuration을 설정하는 과정으로, enable/disable옵션을 이용하여 FFmpeg의 기능을 허용/제한할 수 있다. 특히 enable 옵션을 이용하여 libx264 / libvpx / libopus 등 외부 라이브러리가 필요한 코덱들을 포함시킬 수 있다. 하지만,  --enable-[LIB] 옵션을 사용하기 위해서는 사용하려 하는 [LIB]에 해당하는 라이브러리가 우선 설치가 되어있어야 한다. 설치가 안돼있는 경우 ERROR: [LIB]  not found 메세지와 함께 configuration 과정에 실패하게 된다.

## fftools

FFmpeg source의 fftools의 빌드 결과로는 ffmpeg, ffplay, ffprobe 세 개의 binary가 설치된다. 이 binary들은 FFmpeg의 source 코드를 이용하지 않고도 FFmpeg의 기능을 사용할 수 있게 해준다. 만약 개발 일정이 아주 촉박하거나 multimedia pipeline의 최적화가 중요하지 않은 상황이라면 소스코드에서 fftools를 실행시키는 것도 좋은 방법이 될 수 있다.

### ffmpeg

ffmpeg은 media converting을 지원해주는 binary이다. File / Network / Device로 부터 multi-input을 받을 수 있으며, output 또한 file / network로의 multi-output이 가능하다. 

``` bash
$ ffmpeg [global_options] {[input_file_options] -i input_url} ... {[output_file_options] output_url} ...
```

와 같이 실행할 수 있으며, 다음은 input.mp4 파일을 vp8과 opus를 이용하여 output.webm으로 변환하는 간단한 예제이다.
``` bash
ffmpeg -i input.mp4 -c:v vp8 -c:a opus output.webm
```

ffmpeg binary의 내부에서는 Fig. 1과 같은 pipeline을 만들어서 동작하며, filter 옵션을 통해 decoder와 encoder사이에는 다양한 filter들의 묶음인 filtergraph가 연결시킬 수 있다 . ffmpeg binary에서는 아주 다양한 option들을 제공하고 있으며 **Reference [3]** 문서화 되어있다.

![Pipeline in ffmpeg Binary](/assets/img/post/ffmpeg/overview/pipelin_of_ffmpeg_bin.png)
_Fig. 1: Pipeline in ffmpeg Binary [3]_

### ffplay

ffplay는 FFmpeg lib들과 SDL을 이용하여 만든 간단한 player이다. File과 network로부터 input을 받아올 수 있으며, SDL를 이용하기 때문에 cross-platform에서 재생이 가능하다. ffplay는 다음과 같이 실행이 가능하다.

``` bash
$ ffplay  [options] [input]
```

Network source로 다양한 protocol을 지원하고 있기 때문에, 구현한 내용이  정상동작 하는지 확인하는 용도로 사용하기 적절하다. ffmpeg과 같이 제공하는 옵션들이 FFmpeg 공식 홈페이지를 통해 **Reference [4]**에 문서화 되어있다.

### ffprobe

동영상 파일의 stream혹은 meta정보를 조회할 수 있는 binary이다. 각각의 packet정보까지 제공해주는 옵션도 있기 때문에, 코드를 직접 작성하지 않고도 파일이 정상적인지 확인할 수 있다.

``` bash
$ ffprobe [options] [input_url]
```

와 같이 실행할 수 있으며, 다른 fftools와 마찬가지로 사용가능한 옵션들이 공식 홈페이지의 **Reference [5]**에 문서화 되어있다.

# References
- [0] [https://ffmpeg.org/ffmpeg.html](https://ffmpeg.org/ffmpeg.html)
- [1] [http://ffmpeg.org/doxygen/4.1/index.html](http://ffmpeg.org/doxygen/4.1/index.html)
- [2] [https://github.com/FFmpeg/FFmpeg](https://github.com/FFmpeg/FFmpeg)
- [3] [https://ffmpeg.org/ffmpeg.html](https://ffmpeg.org/ffmpeg.html)
- [4] [https://ffmpeg.org/ffplay.html](https://ffmpeg.org/ffplay.html)
- [5] [https://ffmpeg.org/ffprobe.html](https://ffmpeg.org/ffprobe.html)