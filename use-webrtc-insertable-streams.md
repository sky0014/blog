# 使用 WebRTC Insertable Streams

使用 WebRTC 播放视频时，由于视频流都被封装到了`MediaStream`对象中，如果要分析或处理视频流中的数据（常见的如 SEI 帧）等，以前没有办法，现在我们可以使用`WebRTC Insertable Streams`

参考文档：

[https://htmlpreview.github.io/?https://github.com/guidou/webrtc-media-streams/blob/insertable-streams/index.html](https://htmlpreview.github.io/?https://github.com/guidou/webrtc-media-streams/blob/insertable-streams/index.html)

[https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection)

[https://developer.mozilla.org/en-US/docs/Web/API/TransformStream](https://developer.mozilla.org/en-US/docs/Web/API/TransformStream)

## 使用步骤

1. 启用`Insertable Streams`

   要使用`Insertable Streams`，需要初始化`RTCPeerConnection`时先启用，否则会报错：

   ```
   Failed to execute 'createEncodedStreams' on 'RTCRtpReceiver': Too late to create encoded streams
   ```

   启用方法很简单：

   ```ts
   new RTCPeerConnection({
     encodedInsertableStreams: true, // 启用Insertable Streams
   });
   ```

2. 获取`RTCInsertableStreams`

   使用`createEncodedStreams`方法获取`RTCInsertableStreams`。

   对于发送端，使用的对象是`RTCRtpSender`

   对于接收端，使用的对象是`RTCRtpReceiver`

3. 使用`TransformStream`处理流数据

   下面以接收端为例：

   ```ts
       const peer = new RTCPeerConnection({
           encodedInsertableStreams: true, // 启用Insertable Streams
       });

       peer.addEventListener('track', (event) => {
           const receiver = event.receiver;    // RTCRtpReceiver

           if (receiver.track.kind === 'video') { // 判断是视频
               const { readable, writable } = e.receiver.createEncodedStreams();   // 获取RTCInsertableStreams

               // 定义转换逻辑
               const transformStream = new TransformStream({
                   start() {},
                   transform(chunk, controller) {
                       // 解析webrtc流数据，chunk类型为RTCEncodedVideoFrame，二进制数据是Annex B格式(00 00 00 01隔断)的h264 nalu数据
                       parseNal({ codec: 'h264', bytes: new Uint8Array(chunk.data)});

                       ...

                       controller.enqueue(chunk);
                   },
                   flush() {},
               });

               // 连接流
               readable.pipeThrough(transformStream).pipeTo(writable);
           }
       });

       ...
   ```
