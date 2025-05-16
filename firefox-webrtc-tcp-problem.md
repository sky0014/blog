# Firefox 火狐浏览器中使用 tcp 建立 webrtc 连接的问题

> By sky0014 2025/5/16

最近在做低延迟直播项目，发现在火狐上，使用 webrtc udp 可以正常连接上服务器，但是 tcp 不行（同等条件下 Chrome 正常），对比分析后，发现主要是在 candidate 的处理上，火狐更严格，而 Chrome 可以兼容一些不太标准的格式。

candidate 标准格式（ RFC 5245 ）

```
candidate:<foundation> <component-id> <transport> <priority> <address> <port> typ <candidate-type> [raddr <rel-address>] [rport <rel-port>] [<extension-att-name> <extension-att-value>]
```

我们服务器返回的 candidate(ip 和端口已脱敏)：

```
a=candidate: 1 udp 2000 1.1.1.1 1111 typ host generation 0
a=candidate: 1 tcp 1000 1.1.1.1 1111 typ host generation 0
```

存在 2 个问题：

1. 缺少 foundation，火狐会解析字段失败
1. 对于 tcp 类型的 candidate，缺少 tcptype 字段，火狐无法使用，拉流应该使用`tcptype passive`

修正后：

```
a=candidate:111 1 udp 2000 1.1.1.1 1111 typ host generation 0
a=candidate:111 1 tcp 1000 1.1.1.1 1111 typ host tcptype passive generation 0
```

这样标准化处理之后，火狐上就可以正常使用 webrtc tcp 拉流了。
