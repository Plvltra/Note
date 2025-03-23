https://cs144.github.io/

#### Unit1-nternet-and-IP
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