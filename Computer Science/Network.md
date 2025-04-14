https://cs144.github.io/

#### Unit1 Internet-and-IP
1. 几种典型的网络引用
	-  World Wide Web(HTTP)
		- client-server model, 客户端与服务端建立连接, 客户端HTTP GET请求, 服务端响应
	- BitTorrent
		- peer-to-peer model, 下载一个种子的所有客户端由Tracker介绍，互联成网状，请求对方已下载的部分
	- Skepe即时通讯
		- 前两者的混合模型, 如果客户端位于NAT背后(无法被公网访问), 则Skepe的(Rendezvous/Relay)服务器作为中继
2. 四层网络模型
	- Application(stream of data): 提供两个应用间双向-可靠的byte stream
	- Transport(segments of data): 保证正确性, 顺序, 拥塞控制等
	- Network(packets of data): 再端到端之间传递datagrams, best-effort delivery, (仅IP协议, IP is the "thin waist")
	- Link: 在host-router, router-router之间传递数据
3. IP协议的模型
	- 特点
		- datagram: 是独立自包含的packet， hop-by-hop routing(router根据源IP一跳一跳到达目标IP)
		- unreliable: packet可能丢失,是不可靠的
		- best effort: IP承诺尽最大可能传递(承诺只在必要时丢弃数据包 i.e. 拥塞, 数据包传错, 循环传递...)
		- connectionless: no per-flow state(没有针对每个数据流的状态记录), packet可能乱序到达
	- 设计理念
		- keep simple, dumb,minimal以减少搭建和维护成本
		- 尽可能将feature放在end host实现
		- 对link layer几乎没有假设, 可以搭在丰富的link层上
	- 更多细节
		- 尝试预防packet无线hop循环: tty每次hop减1 
		- 在链路限制下，可以将packet 分片
		- chksum防止传到错误地址
		- 允许IPv4, IPv6
		- 允许header添加新的功能选项
	- IPv4 header各字段的意义说明
4. Life of a Packet
	- router: 记录一个转发表, 根据匹配决定packet的下一hop
5. Packet switching原则
	- 对于每个packet, router根据路由表独立地选择其输出链路(no per-flow state)
	- no per-flow state
		-packet交换不保存流的状态-packet是自包含的，不会因为两个packet属于一个流就记录复用上一个packet转发状态
		- 没有state需要被增/删/存 到router或者失败清理
	- 优点
		- packet转发实现简单, 不需要直到流
		- 可以高效的在不同流之间共享链路
6. Layering原则
	- 模块化
	- 层的服务well-defined
	- 层可以reuse
	- seperation of concerns, 每层只关注自己的功能, 唯一通信是上下层之间
	- continuous improvement, 每层保证协议的情况下可独立迭代
	- peer-to-peer communications
7. Encapsulation原则
	- 封装的灵活性:
		- encapsulation allows to layer recursively: 例如VPN的数据包结构是[ETH [IP [TCP [TLS [IP [TCP [HTTP ]]]]]]],原本的请求嵌套在 TLS
8. 字节序
	- 网络字节序: 大端法
	- 辅助转换函数: htons(),ntohs()
9. IPv4地址
 	- Netmask: 子网掩码, 如果数据包源IP和目标IP经过子网掩码处理相同, 说明处于相同network可以直接发送不需要经过router
	- IP地址结构
		- 分为A,B,C三类，i.e. A类掩码255.0.0.0， 拥有2^24个地址
		- ABC三类划分粒度太大，CIDR可以让掩码数介于之间(例如CIDR前缀的网络掩码可以是10bit)
		- IPv4地址划分: IANA 提供 RIRs(Regional Internet Registries) /8s(A类地址)
10. 链路转发Longest prefix match规则
11. Address Resolution Protocol(ARP)
	- 起因: 由于历史原因, 链路层使用的mac地址和网络层IP地址是分离的。发送数据流, 知道了目标IP还需要知道目标链路层地址mac
	- ARP是局域网广播性质的
	-  示例
		```
		A——————————————————————————————————>gateway———————————————————————————>B
		192.168.0.5		         192.168.0.1	   171.43.22.8		  171.43.22.5
		00:13:72:4c:d9:6a        0:18:e7:f3:ce:la  0:18:e7:f3:ce:1b	  9:9:9:9:9:9
		gateway: 192.168.0.1
		```
		- 如果主机A想给B的IP地址发数据包, 会经过网关。A需要先通过ARP拿到192.168.0.1的mac地址, 发送数据包IP是A和B的, mac是00:13:72:4c:d9:6a和0:18:e7:f3:ce:la
		- 数据包到达gateway后，第二个数据包被封装在IP是A和B的, mac是0:18:e7:f3:ce:1b和9:9:9:9:9:9


#### Unit2 Transport
1. TCP service model
	- TCP在连接的两端维护状态机，以跟踪连接的状态
	- TCP特性
		- 提供可靠的字节传递
			- acknowledgment确保正确传递到
			- checksum保证没有data corrupt
			- sequence numbers detect data missing
			- flow-control 预防了接收方overrunning(超出接收方接收能力)
		- 提供保序的字节传递
		- 拥塞控制
	- TCP Segment 头字段
		- Source port, Destination port: 发送方接收方端口
		- Sequence: 当前数据的序列号
		- Acknowledgment Sequence: 表示已经收到的Segment数和下一个期待的Sequence号
		- Flags
			- ACK(标志Ack Seq是否有效)
			- PSH(当启用此标志, 另一端的TCP层应该立即交付数据)
			- SYN/FIN(我们正在发送同步/结束信号)
	- The Unique ID of a TCP connection
		- (IP DestAddr, IP SrcAddr, IP Protocal ID, TCP DestPort, TCP SrcPort)五元组唯一定义了网络上的一个TCP连接
		- 如果host A短时间创建大量到host B连接, port自增一圈导致创建的两个连接端口相同，如果前一个连接TCP段未发出可能发生混淆(通过随机初始Sequence机制减小了混淆的可能)

2. UDP service model
	- 简单: hdr质保函Source Port，Dest Port, 可选的checksum, Length，通常用于类似DNS这种request-response的情况和少量音视频传输

3. ICMP service model
	- ICMP(The Internet Control Message Protocol): 主要用于报告错误和诊断网络层
	- 可以算作Transport layer, 当要发回错误时，ICMP作为IP Data结合IP的Hdr发回
	- ICMP内容由ICMP Type, ICMP Code, 以及源头方发来的IP Datagram的hdr,data提取出的部分信息组成
	- ping的工作原理: A 发送[[IP], ICMP]给B, B 以同样的[[IP], ICMP]应答
	- traceroute的工作原理: 先发送TTL=1的UDP(带着错误的端口号)到目标IP, 得到第一跳的IP和延迟。再发送TTL=2的,依此类推

4. End to End Principle
	- Network能不能做更多(压缩数据/增加数据加密合校验保证安全性/符合条件自动实现BitTorrent式的下载)? 不行
		- 没有终端的完整需求信息, 仅靠网络无法完整实现需求
		- example: End to End传输文件功能, 以为靠链路层error detection就可以保证数据的完整性。如果传递过程中有台电脑内存有问题，数据加载上去旧损坏。即使链路层保证了transmit的完整性，也没有消除storage的错误。还是要靠End本身收到后校验文件的完整性
	- Network provides incomplete version of the funtion as a performanc enhancement? 可以但不好
		- example: wifi链路层不可靠, 增加确认重传提高可靠性
		- 确实提高了TCP整体传输可靠性, 但是有的传输层可能不需要可靠，并且其他层想要迭代都要wifi链路层重传这个特性。降低了灵活性

5. 三种数据损坏错误检查机制
	- chksum: 高速, 检查错误能力差(IP, TCP层)
	- CRC: 检查错误能力强(Ethrenet层)
	- MAC(message authentication code, TLS层): 主要用来防止恶意篡改, 也可以一定程度上防止错误 

6. 用有限状态机描述TCP连接的状态细节
7. 
	- Flow Control(思想是发送数据不超过接收方处理上限)
		1. Stop  and wait: 简单的保证一次只发一个packet, 超时或未ack重传(在网络延迟情况下要处理ack不准确的问题)
		2. Sliding window: 
	- Reliable comm


8. Sliding window
	- Stop and Wait的问题: 假设一个链路是10Mbps, RTT是50ms, 则最多能传递20个TCP包体, 浪费链路
		- Sliding window是对Stop and Wait(N=1)的泛化
	- Sender
		- 维护3个变量
			- Send window size(SWS)
			- Last acknowledgment received(LAR)
			- Last segment sent(LSS)
		- 维护invariant: (LSS - LAR <= SWS)
		- 收到ack推进LAR(例如收到ack 2、4、5 LAR取2)
		- 缓存最多SWS个segments
	- Receiver
		- 维护3个变量
			- Receive window size(RWS)
			- Last acceptable segment(LAS)
			- Last segment received(LSR)
		- 维护invariant: (LAS - LRR <= RWS)
		- 如果收到的segment < LAS, 发送 ack
			- 发送的是cululative acks， ack segment是下一个期待的segment(例如收到了1,2,3,5, acknowledge 3, 发送ack 4)
	- 滑动窗口概述: 
		- Allow a "window" of unacknowledged packets in flight
		- When acknowledgment arrives, advance window

9. Reliable communication
10. 
11. 
	- Sliding window的两种可选重传策略
		- Go-back-N(N=滑动窗口大小): one loss will lead to entire window retransmitting
			- 优点: 如果网络发生明显的错误如连续丢失多个包体, 重传耗时更少(不像selective repeat需要逐个执行time out重传)
			- 使用场景: 例如发送端SWS(Send window size, 假设4个包大小)远大于RWS(假设1个包大小)。若使用selective port只补传loss的包, loss后续的包也会因为接收方缓存不足而被丢弃。此时Go-back-N更优
		- Selective repeat: one loss wille lead to only that packet retransmitting 
			- 优点: 发送的数据量更少
			- 使用场景: 例如发送端SWS(假设4个包大小)接近于RWS(假设1个包大小), 如果1号包丢失, 2、3已经被接收方缓存, 4因为滑动窗口已经发送, 只需要重传1就好了
	- TCP Header补充
		- sequence number/acknowledgment number: 单位是字节, 表示发送的序列号/已接收字节的下一个期待字节
		- window: 单位是字节, 表示接收方当前可用的缓冲区大小(用于滑动窗口接收方大小计算)
		- u flags和urgent pointer: u表示这个TCP数据包需要立即被应用层处理, urgent pointer指向需要处理的数据
	- Connnection setup and teardown
		- Problems:
			- final ack可能丢失
			- 相同端口被立即重用于新的连接可能会有空间overlapping的问题
		- Solution: 
			- 主动关闭者TIME WAIT一段时间后关闭: keep socket around for 2MSL(maximum segment lifetime)
			- 服务器作为主动关闭者可能会出现过多端口TIME WAIT
				- 设置RST删除socket
				- 可以设置SO_LINGER选项为0使得停留时间为零
				- SO_REUSEADDR绕过OS限制重新绑定端口


#### Unit3 Packet Switching
1. History of Networks/Internets
2. What is Packet Switching
	- Circuit Switching
		- 例如电话呼叫连接, Each call has its own private, guaranteed, isolated data rate from end-to-end
		- A call has three phases(establish, communicate, tear down)
		- Originally, a circuit was an end-to-end physical wire(nowadays, is virtual private wire)
		- Problems:
			- Inefficient(网络连接时不间断, 不是持续的)
			- Diverse rates()
			- State Management(Circuit switches维护per-communication状态)
	- Packet Switching
		- Packet根据router's local table独立路由
		- All packets share the full capacity of a link
		- The routers不维护per-communication state.
		- Packet switches have buffers(switch同时只能发送一个packet, 如果来的太多或者发生拥塞需要buffer)
	- Why Internet use Packet Switching?
		- Efficient use of expensive links(允许许多，同时的flows共享相同的link)
		- Resilience to failure of links & routers(Even routers were destroyed, messaget could be rerouted.)

3. End-to-end delay & Queuing delay
	- 一些定义
		- Propagation Delay: 1 bit在一段链路上传播的用时(t = l / c, l: 链路长度, c: 光速)
		- Packetization Delay: 1个packet从第一个bit到最后一个bit被加载上链路所花的时间(t = p / r,  p: packet大小, r: bit流加载到link的速率 bit/s)
	- End-to-end delay & Queuing delay
		- Queuing delay: 因packet途径router的buffer中缓存了packet产生的延迟
		- End-to-end delay: t = packet途径每一段的(Propagation Delay + Packetization Delay + Queuing delay)累加和
	- Ping工具: 记录end-to-end一个来回的延迟时长

4. Playback buffers
	- Some real-time app use playback buffers to absorb the variation in queueing delay, such as a video player	
	- With packet switching, end-to-end delay is variable. Applications estimate the delay, set the playback buffer big enough
	![[PlaybackBuffers.png]]

5. Queue models
	- Simple deterministic queue model
	- 可以简化为队列辅助理解packet在转发过程中的堆积: 多个进入link依次向router写入packet, 一个出的link将packet依次发送出
	![[SimpleQueueModel.png]]
	- Small packets reduce end to end delay
	- 增加了传输的并行度, 减少了端到端延迟

6. Queuing model properties
	- burstiness increases delay, determinism minimizes delay
	- Little's Result: L = λd
	- L: average number of customers queuing in the system(average number of packet bytes)
	- λ: arrival rate per second
	- d: average time that a customer waits in the system
	- M/M/1 queue(Poisson process)

7. Switching and forwarding(1, looked up in the forwarding table)
	- A packet switch look like:
		![[GenericPacketSwitch.png]]
	- What does a packet switch do?
		- Ethernet switch
		- 检查到达frame的头部
		- 如果Ethernet DA在 forwarding table, 转发到对应端口。
		- 如果不在, 向所有端口广播
		- forwarding table会在检查收到packet的SA(source address)自学习
	- Internet router
		- 如果the Ethernet DA of the arriveing frame与当前router相等则接收，否则丢弃
		- 检查IP version和datagram长度, 减少TTL更新IP header checksum, 检测TTL=0
		- 如果IP DA在forwarding table，forward to the correct egress port for the next hop
		- 找到next hop router的ethernet DA(没有则使用ARP), 创建新的Ethernet frame并发送
	- Lookup Address
		- Ethernet switch: 存储在 hash table, 根据mac地址精确匹配
		- Internet router: Lookup是longest prefix match, 而非精确匹配
			- 实现方式1: 建立binary tries前缀树匹配
			- 实现方式2: Ternary Content Addressable Memory (TCAM)
	- At a high level, (switches, routers, firewalls)等lookups可以可以泛化成: <Match, Action>的形式

8. Switching and forwarding(2, forwarding: Send packet to the correct output port))
	- Output queued packet switch
		![[IntputQueuedPacketSwitch.png]]
		- Lookup 处理后直接扔给Ouput, 由Output负责对packet queue逐个发送
		- 优点: delay能达到所有队列模型最理想的情况
		- 缺点: scalability很差(不能无限加lookup管线), Output queue的内存可能存在(n+1) * r的压力, n(n个lookup) 以r(rate速率)写入, 1个写出
	- Input queued packet switch
	![[IntputQueuedPacketSwitch.png]]
	- 每个Loopup处理后如果输出端口空闲就发送, 如果输出端口被占用就queued住
		- with Head of Line Blocking
			- 可能会出现头部的一个packet出不去, 影响后面的packet	
		- with Virtual output queues
			- 吞吐量最大化	
			- 优点: 实现上不会出现单个Output queued的内存压力，router的可扩展性强
	- The simplest and slowest switches use **output queueing**, which minimizes packet delay.
	- High performance switches often use **input queueing, with virtual output queues** to maximize throughput.

9. Rate guarantees