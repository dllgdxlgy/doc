## 网络体系结构

![计算机网络体系结构](https://tva1.sinaimg.cn/large/00831rSTly1gd9qb100evj31f40ocdvs.jpg)

| 分层       | 作用                                                         | 主要协议                                                     |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 应用层     | 通过应用进程间的交互来完成特定网络应用。                     | DNS、HTTP、SMTP、FTP                                         |
| 表示层     | 定义信息之间的语法语义及其关联，如加密解密、转换翻译、压缩与解压缩。 | XDP                                                          |
| 会话层     | 为不同及其上的用户之间建立及管理会话。                       | SSL、TLS、RPC                                                |
| 运输层     | 接受上一层的数据，在必要的时候进行分割，并将这些数据交给网络层，且保证这些数据段有效到达对端。 | TCP、UDP                                                     |
| 网络层     | 负责为分组交换网上的不同主机提供通信服务。控制子网的运行，如逻辑编址、分组传输、路由选择。 | SLIP（串行线路 IP 协议）、IP、ICMP、IGMP、EGP、GGP、RIP（路由信息协议）、OSPF（开放最短路径优先协议） |
| 数据链路层 | 将网络层交付的IP数据包封装成帧，负责物理寻址，同时将原始比特流转换为逻辑传输线路。 | ARP、RARP、                                                  |
| 物理层     | 尽可能屏蔽传输媒体和通信手段的差异，透明地传输比特流。       | IEEE802.2、Ethernet v.2                                      |

## 常见的使用 TCP 与 UDP 的应用层服务

- 使用 TCP：HTTP、FTP（文件传输协议）、SMTP（简单邮件传输协议）、TELNET（终端仿真协议）、POP3（邮局协议第三版）、IMAP4。
- 使用 UDP：TFTP（简单文件传输协议）、DHCP（动态主机配置协议）。
- 使用 TCP 与 UDP：SOCKS（安全套接子协议）、DNS（域名系统）

## 三次握手

![三报文握手建立连接](https://tva1.sinaimg.cn/large/00831rSTly1gd9tsfxtkyj31ce0okdrn.jpg)

1. A 主动打开连接，B 被动打开连接。
2. B 的 TCP 服务器进程 创建 传输控制块TCB，准备接受客户进程的连接请求，进入监听(Listen)状态。
3. A 的 TCP 客户端进程 创建 传输控制块 TCB。
4. A 向 B 请求建立连接时，发出连接请求报文段，首部中同步位 SYN=1，选择初始序号 seq=x。TCP客户进程进入SYN-SENT(同步已发送)状态。
5. B收到连接请求报文段后，如同意建立连接，则向A发送确认。在确认报文段中携带 SYN=1，ACK=1，ack=x+1，seq=y。服务器进程进入SYN-RCVD(同步收到)状态。
6. TCP客户进程收到 B 的确认后，向B发出确认。携带 ACK=1，ack=y+1，seq=x+1。TCP连接已经建立，A 进入 ESTABLISHED (已建立连接)状态。
7. 当 B 收到 A 的确认后，也进入 ESTABLISHED 状态。

> 问：为什么 A 最后还要发送一次确认呢？（为什么需要三次握手？两次不行吗？）
> 答：这主要是为了防止已失效的连接请求报文段突然又传送到了B，因而产生错误。
> 如果只有两次握手的话，那么只要 B 发出确认，新的连接就建立了。由于现在 A 并没有发出建立连接的请求，因此不会理睬 B 的确认，也不会向B发送数据。但 B 却以为新的运输连接已经建立了，苦苦地等待 A 发来数据，B 的许多资源就这样白白浪费了。

## 四次挥手

![TCP连接的释放过程](https://tva1.sinaimg.cn/large/00831rSTly1gd9uuv1g7cj31ew0r2qje.jpg)

1. 在关闭连接前，A 和 B 都处于 ESTABLISHED 状态。
2. **A 的应用进程**先向其 TCP 发出连接释放报文段，并停止发送数据，主动关闭 TCP 连接。连接释放报文段首部终止控制位 FIN=1，seq=u，它等于前面已传送过的数据的最后一个字节的序号加 1。A进入 **FIN-WAIT-1(终止等待1)** 状态，等待 B 的确认。
3. B 收到连接释放报文段后即发出确认，确认位 ACK=1，确认号是 ack=u+1，序列号seq=v，等于 B 前面已传送过的数据的最后一个字节的序号加 1。
   1. 进入 **CLOSE-WAIT(关闭等待)** 状态。
   2. TCP 服务器进程通知高层应用进程，此时从 A 到 B 这个方向的连接已释放，TCP连接处于 **半关闭(half-close)** 状态，即 A 已经没有数据要发送了，但 **B 若发送数据，A 仍要接收**。也就是说，从B到A这个方向的连接并未关闭，这个状态可能会持续一段时间。
4. A 收到 B 的确认后，进入 **FIN-WAIT-2(终止等待2)** 状态，等待 B 发出的连接释放报文段。
5. 若 B 已经没有要向A发送的数据，其应用进程就通知 TCP 释放连接。
   1. B 发出的连接释放报文段中终止位 FIN=1，确认位 ACK=1，序列号 seq=w (半关闭状态下 B 可能又发送了一些数据)。
   2. B 还必须**重复上次已发送过的确认号 ack=u+1**。B进入 **LAST-ACK(最后确认)** 状态，等待A的确认。
6. A 收到 B 的连接释放报文段后，对此发出确认。
   1. 在确认报文段中，ACK=1，ack=w+1，seq=u+1，然后进入 **TIME-WAIT(时间等待)** 状态。
   2. 此时 **TCP连接还没有释放掉**，必须经过 **2MSL** 后，A才进入到 **CLOSED状态**。
   3. MSL叫做最长报文段寿命(Maximum Segment Lifetime)，RFC793建议设为2分钟。但 TCP 允许不同实现自定义。因此，从A进入到 **TIME-WAIT状态**后，要经过 **2MSL** 才能进入到 **CLOSED 状态**，才能开始建立下一个新的连接。
   4. 当A撤销相应的传输控制块TCB后，就结束了这次的TCP连接。
7. B 收到 A 发出的确认后，才进入 **CLOSED状态**，在 B 撤销 PCB 后，本次 TCP 连接才算结束。

> TIME-WAIT 为什么要等待 2MSL？
>
> - 第一，为了保证 A 发送的最后一个 ACK 报文段能够到达 B。这个 ACK 报文段有可能丢失，因而使处在 LAST-ACK 状态的 B 收不到对已发送的 FIN+ACK 报文段 的确认。B 会超时重传这个 FIN+ACK 报文段，而 A 就能在 2MSL 时间内收到这个重传的 FIN+ACK 报文段。接着 A 重传一次确认，重新启动 2MSL 计时器。最后，A 和 B 都正常进入到 CLOSED 状态。**如果 A 在 TIME-WAIT 状态不等待一段时间，而是在发送完 ACK 报文段后立即释放连接，那么就无法收到 B 重传的 FIN+ACK 报文段，因而也不会再发送一次确认报文段。**这样，B就无法按照正常步骤进入 CLOSED 状态。
>
> - 第二，防止“已失效的连接请求报文段”出现在本连接中。A 在发送完最后一个 ACK 报文段后，再经过时间 2MSL，就可以使本连接持续的时间内所产生的所有报文段都从网络中消失。这样就可以使下一个新的连接中不会出现这种旧的连接请求报文段。

## CLOSE-WAIT 与 TIME-WAIT 作用及阶段？

- CLOSE-WAIT：被动关闭连接一方在收到主动关闭连接一方的关闭请求，并对此作出确认之后。
- TIME-WAIT： 主动关闭连接一方在收到被动关闭连接一方的连接释放报文段，并对此发出确认之后。

## RIP 协议怎么解决的环路问题？

1. 最大跳数：路由达到16跳就认为不可达。

2. 水平分割：不向原始路由更新来的方向再次发送路由更新信息。

3. 路由毒化：先检测到网络不可达的路由器向周围路由器扩散这个消息。

4. 反向毒化：如果A路由器告诉了B路由器 C路由器不可达，B路由器再反过来告诉A路由器一次。保证所有路由器都收到了毒化的路由信息。

5. 控制更新时间：当路由表中的某个条目所指网络消失时，路由器并不会立刻的删除该条目并学习新条目，而是严格按照我们前面所介绍的计时器时间现将条目设置为无效接着是挂起，在240秒时才删除该条目，这么做其实是为了尽可能的给予一个时间等待发生改变的网络恢复。

6. 触发更新：因网络拓扑发生变化导致路由表发生改变时，路由器立刻产生更新通告直连邻居，不再需要等待30秒的更新周期，这样做是为了尽可能地将网络拓扑的改变通告给其他路由器。

## TCP 是如何保证可靠传输的？

## TCP 拥塞控制与流量控制

## QQ 是使用的 TCP 还是 UDP？

## 粘包问题

## IP 地址划分

## ARP 与 RARP