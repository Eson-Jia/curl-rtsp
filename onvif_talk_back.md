# onvif talk back

## 前言

在视频监控应用场景中，有很多需要采集查看监控的人的语音发送到监控摄像头的音响上去以实现对被监控对象的语音控制。
`ONVIF Core Specification Core_2.00文档`中章节`12.3 Back Channel Connection`对此进行了详细的描述。
`ONVIF`语音对讲的实现完全基于`RTSP`协议，流程中没有用到`ONVIF`协议。

## RTSP Require-Tag

RTSP 标准[RFC 2326]可以通过添加额外的头进行扩展，`Require`tag被引入用于处理特殊的功能扩展(参考 [RFC
2326], 1.5 Extending Rtsp and 12.32 Require).
`Require`头用来判定是否支持某个特性，如果要求`server`理解某个特性并正确处理请求，需要对`server`的每个请求都携带这个`Require`头。
`server`如果支持`backchannel`这个特性，需要理解`backchannel`对应的tag:`www.onvif.org/ver20/backchannel`

## DESCRIBE

在`Client - Server`发送`DESCRIBE`协议的时候添加前文说过的`Require-tag`,这是如果`Server`不支持语音对讲则会回复`551 Option not supported`，示例如下：

```bash
Client – Server: DESCRIBE rtsp://192.168.0.1 RTSP/1.0
CSeq: 1
User-Agent: ONVIF Rtsp client
Accept: application/sdp
Require: www.onvif.org/ver20/backchannel #!! Require-tag
Server – Client: RTSP/1.0 551 Option not supported
CSeq: 1
Unsupported: www.onvif.org/ver20/backchannel
```

如果支持语音对讲的话，则会回复`200 OK`并携带`sdp`信息:

```bash
RTSP/1.0 200 OK
CSeq: 1
Content-Type: application/sdp
Content-Length: xxx
v=0
o= 2890842807 IN IP4 192.168.0.1
s=RTSP Session with audiobackchannel
m=video 0 RTP/AVP 26
a=control:rtsp://192.168.0.1/video
a=recvonly
m=audio 0 RTP/AVP 0
a=control:rtsp://192.168.0.1/audio
a=recvonly
m=audio 0 RTP/AVP 0
a=control:rtsp://192.168.0.1/audioback
a=rtpmap:0 PCMU/8000
a=sendonly
```

上面的`sdp`列出了三个流及其控制`URL`: 视频流：`rtsp://192.168.0.1/video`,音频流:`rtsp://192.168.0.1/audio`,以及我们的主角对讲流`rtsp://192.168.0.1/audioback`,注意对讲流的属性**a=sendonly**与其他流的**a=recvonly**不同。

## SETUP

接下来我们就可以`SETUP`这些`session`:

```rtsp
Client – Server: SETUP rtsp://192.168.0.1/video RTSP/1.0
CSeq: 2
Transport: RTP/AVP;unicast;client_port=4588-4589
Server – Client: RTSP/1.0 200 OK
CSeq: 2
Session: 123124;timeout=60
Transport:RTP/AVP;unicast;client_port=4588-4589;
server_port=6256-6257
Client – Server: SETUP rtsp://192.168.0.1/audio RTSP/1.0
CSeq: 3
Session: 123124
Transport: RTP/AVP;unicast;client_port=4578-4579
Server – Client: RTSP/1.0 200 OK
CSeq: 3
Session: 123124;timeout=60
Transport:RTP/AVP;unicast;client_port=4578-4579;
server_port=6276-6277
Client – Server: SETUP rtsp://192.168.0.1/audioback RTSP/1.0
CSeq: 4
Session: 123124
Transport: RTP/AVP;unicast;client_port=6296-6297
Require: www.onvif.org/ver20/backchannel
Server – Client: RTSP/1.0 200 OK
CSeq: 4
Session: 123124;timeout=60
Transport:RTP/AVP;unicast;client_port=6296-6297;
server_port=2346-2347
```
上面`setup`了三次，分别建立了视频流，音频流以及最后一个的音频对讲流的连接。

## PLAY

由于`rtsp`有集合控制的功能，仅需要发送一条`PLAY`或者`PAUSE`就可以同时控制多个音频流和视频流。所以下面我们发送一条`PLAY`请求即可:

```bash
Client – Server: PLAY rtsp://192.168.0.1 RTSP/1.0
CSeq: 5
Session: 123124
Require: www.onvif.org/ver20/backchannel
Server – Client: RTSP/1.0 200 OK
CSeq: 5
Session: 123124;timeout=60
```

在收到`PLAY`请求的`200 OK`的回复之后，客户端就**可以**向`Server`发送音频数据包了，`Client`不应该在收到回复之前就开始发送数据包。
上面例子中的`Require: www.onvif.org/ver20/backchannel`头指明了这个特殊的解释对于`PLAY`命令是必须的。`PLAY`命令同时开始了`Client`从`NVT`获取音频流和视频流以及从`Client`发送到`Server`的对讲流。

## TEARDOWN

通过发送`TEARDOWN`请求来关闭这个`session`:

```bash
Client – NVT: TEARDOWN rtsp://192.168.0.1 RTSP/1.0
CSeq: 6
Session: 123124
Require: www.onvif.org/ver20/backchannel
NVT – Client: RTSP/1.0 200 OK
CSeq: 6
Session: 123124
```

## Multicast Streaming

如果客户端试图使用多播来发送数据，那么它需要使用`SETUP`请求中的`transport`参数来告诉`Server`多播地址和端口

```bash
Client – Server: SETUP rtsp://192.168.0.1/audioback RTSP/1.0
CSeq: 4
Session: 123124
Transport:RTP/AVP;multicast;destination=224.2.1.1;port=60
000-60001;ttl=128
Require: www.onvif.org/ver20/backchannel
Server – Client: RTSP/1.0 200 OK
CSeq: 4
Session: 123124;timeout=60
Transport:RTP/AVP;multicast;destination=224.2.1.1;port=60
000-60001;ttl=128;mode=”PLAY”
```