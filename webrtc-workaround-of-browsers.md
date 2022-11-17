# Web 各版本浏览器之间的 WebRTC 兼容问题及解决方案

不通浏览器版本间的 WebRTC API 兼容性问题可以使用 webrtc-adapter 解决（注意库版本和兼容到的浏览器版本间的对应关系），不在本文讨论之列：
[https://github.com/webrtcHacks/adapter](https://github.com/webrtcHacks/adapter)

解决下列问题需要修改 sdp，可以使用这个库：
[https://www.npmjs.com/package/sdp-transform](https://www.npmjs.com/package/sdp-transform)

## session_id 超过 int64 最大值问题

1. 客户端错误提示

```
OperationError:SIPCCFailedtoparseSDP:SDPParseErroronline2:Invalidownersessionidspecifiedforo=.\n,errCode=ERR_SET_REMOTE_DESCRIPTION
```

2. 错误原因分析

标准[rfc3264](https://datatracker.ietf.org/doc/html/rfc3264)规定：owner session_id 是有符号的 64 位整数，但部分老版本 chrome 浏览器该字段数值超过了有符号 64 位整数最大值，火狐浏览器解析会报错。

现在新的 chrome 中已修复这个问题

参考网上问题：
[https://bugzilla.mozilla.org/show_bug.cgi?id=861895](https://bugzilla.mozilla.org/show_bug.cgi?id=861895)

3. 解决方案

p2p 使用 webrtc 的场景中，sdp 的 session_id 没有作用，可以修改为固定数值 1。

```js
let sdp = "some sdp...";
const obj = sdpTransform.parse(sdp);
obj.origin.sessionId = 1;
sdp = sdpTransform.write(obj);
```

## 新旧 sdp 格式问题

常见的两种格式 sdp，老格式：

```
v=0
o=- 6979679425931439850 2 IN IP4 127.0.0.1
s=-
t=0 0
m=application 9 UDP/DTLS/SCTP 5000
c=IN IP4 0.0.0.0
a=ice-ufrag:qa9t
a=ice-pwd:rl7Umb5EhKEllk2C346i1wdm
a=ice-options:trickle
a=fingerprint:sha-256 39:05:AB:6D:BA:E5:B5:31:D0:4D:99:B6:3C:C9:6F:59:63:CE:D5:28:C5:76:5C:B5:A0:4E:E5:7C:5A:5A:1E:E2
a=setup:active
a=mid:0
a=sctpmap:5000 webrtc-datachannel 1024
```

新格式：

```
v=0
o=- 4623470282706969322 2 IN IP4 127.0.0.1
s=-
t=0 0
a=group:BUNDLE 0
a=extmap-allow-mixed
a=msid-semantic: WMS
m=application 9 DTLS/SCTP webrtc-datachannel
c=IN IP4 0.0.0.0
a=ice-ufrag:IqQG
a=ice-pwd:mmBEbZsGs73noxDfWOGYtoy1
a=ice-options:trickle
a=fingerprint:sha-256 16:3C:BB:46:03:23:8E:97:1E:B8:99:B6:D4:BD:DF:64:80:E7:2E:CB:0E:00:FC:EF:E2:15:2C:5B:E4:68:C6:56
a=setup:active
a=mid:0
a=sctp-port:5000
a=max-message-size:262144
```

根据线上数据估计，Chrome75（包含）之前都是用的老格式，之后用的新格式。

新版本 chrome 同时支持新旧两种格式，但是新版火狐只支持新格式。

同时，有部分属性，比如 a=extmap-allow-mixed，部分旧版本浏览器会解析出错。

解决办法：

1. 检测浏览器是使用的新格式还是旧格式
2. 将收到的 sdp 转换位浏览器支持的格式
3. 去除部分可能解析出错的属性

```js
// 检测浏览器使用的新旧sdp格式
pc.createOffer().then((offer) => {
  if (offer.type === "offer") {
    const sdp = offer.sdp;
    const obj = sdpTransform.parse(sdp);
    isSDPOldFormat = !!obj.media[0].sctpmap;
    logger.log(`sdp old format: ${isSDPOldFormat}`);
  }
});

// 新转旧
const obj = sdpTransform.parse(sdp);
obj.media[0].sctpmap = {
  app: obj.media[0].payloads,
  maxMessageSize: 1024,
  sctpmapNumber: obj.media[0].sctpPort,
};
obj.media[0].payloads = `${obj.media[0].sctpPort}`;
delete obj.media[0].sctpPort;
obj.media[0].protocol = "DTLS/SCTP";

// 旧转新
const obj = sdpTransform.parse(sdp);
obj.media[0].sctpPort = +obj.media[0].sctpmap.sctpmapNumber;
obj.media[0].payloads = obj.media[0].sctpmap.app;
obj.media[0].protocol = "UDP/DTLS/SCTP";
delete obj.media[0].sctpmap;

// 过滤无用属性
const obj = sdpTransform.parse(sdp);
delete obj.extmapAllowMixed;
delete obj.msidSemantic;
delete obj.groups;
```

## fingerprint 和 iceOptions 位置问题

老版本火狐浏览器，fingerprint 和 iceOptions 属性会设置到上面，没有跟 ice-pwd 等属性设置到一起，设置到上面其实是全局的意思，但是 native sdk 不支持全局属性，因此 sdk 会认为没有 fingerprint 等属性，从而返回一个不带加密的 sdp 回来，造成浏览器端无法成功建立连接。

解决方案：

web 端进行兼容，将 fingerprint 和 iceOptions 属性移到下面。

```js
const obj = sdpTransform.parse(sdp);

if (obj.fingerprint) {
  obj.media[0].fingerprint = obj.fingerprint;
  delete obj.fingerprint;
}

if (obj.iceOptions) {
  obj.media[0].iceOptions = obj.iceOptions;
  delete obj.iceOptions;
}

sdp = sdpTransform.write(obj);
```
