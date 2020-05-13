# gtreamer

## gst-inspect

`gst-inspect pulse`输出的`Element Properties`列表中，`Properties`有不同的`flags`：`readable,writable`.
如果只有`readale`说明这是一个只读属性，不能传入相应的值，否则我们可以通过 `property=value`传入参数。

## gst-launch

`pulsesrc`中`device`是可读可写的，`current-device`是只读的。

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

可以看出`pulsesrc`使用的输入设备是`alsa_input.usb-Generic_ICT_Camera_200901010001-02.analog-stereo`，`autoaudiosink`使用的输出设备是`alsa_output.pci-0000_02_02.0.analog-stereo`。

## pulseaudio

- pulseaudio 是一个 server
- pactl 是一个 pulseaudio 的工具
  - pactl list sources 列出所有输入设备
    - Name alsa_input.pci-0000_02_02.0.analog-stereo
      - State: SUSPENDED
    - Name alsa_input.usb-Generic_ICT_Camera_200901010001-02.analog-stereo
      - State: SUSPENDED
