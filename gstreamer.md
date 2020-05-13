# gtreamer

## gst-inspect

`gst-inspect pulse`输出的`Element Properties`列表中，`Properties`有不同的`flags`：`readable,writable`.
如果只有`readale`说明这是一个只读属性，不能传入相应的值，否则我们可以通过 `property=value`传入参数。

## gst-launch

`pulsesrc`中`device`是可读写的，`current-device`是只读的。

运行命令`gst-launch-1.0 pulsesrc -v ! audioconvert ! autoaudiosink`得到以下输出：

```shell
...
省略多行
...
/GstPipeline:pipeline0/GstPulseSrc:pulsesrc0: current-device = alsa_input.usb-Generic_ICT_Camera_200901010001-02.analog-stereo
/GstPipeline:pipeline0/GstAutoAudioSink:autoaudiosink0/GstPulseSink:autoaudiosink0-actual-sink-pulse: volume = 1
/GstPipeline:pipeline0/GstAutoAudioSink:autoaudiosink0/GstPulseSink:autoaudiosink0-actual-sink-pulse: mute = false
/GstPipeline:pipeline0/GstAutoAudioSink:autoaudiosink0/GstPulseSink:autoaudiosink0-actual-sink-pulse: current-device = alsa_output.pci-0000_02_02.0.analog-stereo
...
省略多行
...
```

注意输出`pulsesrc`Element `current-device = alsa_input.usb-Generic_ICT_Camera_200901010001-02.analog-stereo`以及
`autoaudiosink current-device = alsa_input.usb-Generic_ICT_Camera_200901010001-02.analog-stereo`

## pulseaudio

- pulseaudio 是一个 server
- pactl 是一个 pulseaudio 的工具
  - pactl list sources 列出所有输入设备
    - Name alsa_input.pci-0000_02_02.0.analog-stereo
      - State: SUSPENDED
    - Name alsa_input.usb-Generic_ICT_Camera_200901010001-02.analog-stereo
      - State: SUSPENDED

## 采集 USB 相机音频并发送相机

完整命令：

```bash
gst-launch-1.0  pulsesrc ! audioconvert ! audio/x-raw,channels=1 ! audioresample ! audio/x-raw,rate=44100 ! avenc_aac ! aacparse ! audio/mpeg,mpegversion=4,stream-format=adts ! rtpgstpay pt=104 ! udpsink host=192.168.10.147 port=8236 bind_port=1234
```
