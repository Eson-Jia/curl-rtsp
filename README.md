# ONVIF 语音对讲实现流程

要实现语音对讲，我们需要:

1. 使用携带特定`Require`头的`RTSP`协议与相机通讯，获取`SDP`文件，然后建立`backchannel`连接
2. 采集电脑的音频
3. 封装音频并通过网络发送给相机

## RTSP 部分

参考[ONVIF 文档](./onvif_talk_back.md)使用特殊的`Require`头进行一系列请求之后我们就获得了完整[SDP](./support-backchannel.sdp),这里重点显示`backchannel`的那部分：

```bash
m=audio 0 RTP/AVP 104
c=IN IP4 0.0.0.0
b=AS:50
a=sendonly
a=control:rtsp://192.168.10.147/trackID=4
a=rtpmap:104 mpeg4-generic/16000/1
a=fmtp:104 profile-level-id=15; streamtype=5; mode=AAC-hbr; config=1408;SizeLength=13; IndexLength=3; IndexDeltaLength=3; Profile=1;
a=Media_header:MEDIAINFO=494D4B48010200000400000101200110803E0000803E000000000000000000000000000000000000;
a=appversion:1.0
```

由上面可以看出，媒体类型`audio`,传输协议是`RTP/AVP`,`RTP payload`是104,属性`sendonly`表明只接收数据，控制流的`uri`是`rtsp://192.168.10.147/trackID=4`。属性`rtpmap`中`rtp payload`是104,媒体格式为`mpeg4-generic`也就是`aac`，采样率为`16000`，通道数为`1`。知道这些信息之后我们就可以`SETUP`建立连接了。
发送`SETUP rtsp://192.168.10.147/trackID=4`请求，其中`Transport`设为`RTP/AVP;unicast;client_port=1234-1235`,请求与回复如下：

```http
SETUP rtsp://192.168.10.147/trackID=4 RTSP/1.0
CSeq: 4
Session: 123124
Transport: RTP/AVP;unicast;client_port=1234-1235
Require: www.onvif.org/ver20/backchannel
RTSP/1.0 200 OK
CSeq: 4
Session: 123124;timeout=60
Transport:RTP/AVP;unicast;client_port=1234-1235;server_port=8372-8373
```

然后根据回复中的`server_port=8372-8373`可以得知服务端分别使用`8372`接受`RTP`,`8373`接受`RTCP`。
在完成`PLAY`请求之后我们就可以开始推送电脑采集的音频流了。

## 采集电脑的音频

要想推送电脑的音频流首先需要采集电脑的音频流，在`gtreamer`的`Element`中`pulsesrc`可以非常容易地获取到[pulseaudio](https://www.freedesktop.org/wiki/Software/PulseAudio/)服务中的音频，例如:

- 采集并播放音频：`gst-launch-1.0 pulsesrc ! audioconvert ! autoaudiosink`
- 采集音频并编码成带`adts`的`aac`格式并保存文件：`gst-launch-1.0 pulsesrc ! avenc_aac ! aacparse ! audio/mpeg,mpegversion=4,stream-format=adts ! filesink location=colletion.aac`
  - acenc_aac 将采集的`audio/x-raw`编码成`audio/mpeg,mpegversion=4,stream-format=raw`
  - aacparse 将`aac`由`stream-format=raw`转成`stream-format=adts`也就是添加了`adts(Audio Data Transport Stream)`头
  - filesink 将数据保存成文件

## 封装成 RTP 并发送到相机

我们已经采集到音频流，最后只需要封装成`RTP`并通过`udp`发送到相机的`UDP`端口`8372`就行了,使用如下命令：

- `gst-launch-1.0 pulsesrc ! avenc_aac ! aacparse ! audio/mpeg,mpegversion=4,stream-format=adts ! rtpgstpay pt=104 ! udpsink host=192.168.10.147 port=8372 bind_port=1234`
  - rtpgstpay 将带`adts`头的`aac`封装成`rtp`包,其中`pt`就是`payload type`用的是上面指定的`104`
  - udpsink 把`rtp`包使用`udp`协议从本机的`1234`端口发送到`192.168.10.147:8372`
这时候我们在电脑麦克上说话就可以在相机的音响上播放出来了。
