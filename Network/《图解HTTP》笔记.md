## 一、Web及网络基础

### 1.1 TCP/IP 协议

TCP/IP 是互联网相关的各类协议族的总称，按层次分为：

- 应用层：应用层决定了向用户提供应用服务时通信的活动。HTTP, FTP, DNS 协议之类的处于这一层
- 传输层：提供两台计算机之间的数据传输，TCP, UDP 协议处于这一层
- 网络层：处理在网络上流动的数据包，IP 协议处于这一层
- 链路层（数据链路层）：处理网络的硬件部分，网卡、光纤等物理可见的部分。

### 1.2 TCP/IP 通信传输流

![image-20210303171045803](C:\Users\62566\AppData\Roaming\Typora\typora-user-images\image-20210303171045803.png)

利用TCP/IP 协议族进行通信时，会通过分层顺序与对方进行通信，发送端从应用层往下走，接收端则从应用层往上走。

发送端在层与层之间传输数据时，每经过一层时必定会被打上一个该层所属的首部信息，反之，接收端在层与层之间传输数据时，每经过一层时会把对应的首部消去。 这种把数据信息包装起来的做法称为**封装**。

### 1.3 与HTTP关系密切的协议： IP、TCP和DNS

#### 1.3.1 IP协议——负责传输

IP协议（这里的IP协议并不是指IP地址，位于网络层）的作用是把数据包传送给对方，而要保证确实送到对方那里，则需要知道目标的IP地址和MAC地址（MAC地址指的是网卡地址，IP地址可变，但是MAC地址是不变的。根据APR协议就能通过IP地址定位到对应的MAC地址）。

- IP：将数据包传输给对方，两个重要条件：IP地址和MAC地址。
  - IP地址指明了节点被分配到到地址，MAC地址是指网卡所属固定地址。IP地址可以和MAC地址进行配对。IP地址可以变换，但MAC地址基本上不会更改。

- ARP协议凭借MAC地址进行通信：网络上两台计算机进行通信，通常要经过多台计算机和网络设备中转才能连接到对方。在中转时，会利用下一站中转设备的MAC地址来搜索下一个中转目标。这时采用ARP（地址解析协议），根据通信放的IP可以反查出对应MAC地址。

#### 1.3.2  TCP协议：将大块数据分割以报文段为单位进行数据传输。

- 字节流服务：为了方便传输，将大块数据分割成以报文段（segment）为单位的数据包
- 可靠：利用三次握手策略（three-way handshaking）确保数据能到达目标

#### 三次握手

TCP不会对传送后的情况置之不理，他一定会向对方确认是否成功到达。握手过程中使用了TCP的标志--- SYN(synchronize)和ACK（acknowleagement）。

1. 发送端首先发送一个带SYN标志的数据包给对方。
2. 接收端收到后，回传一个SYN/ACK标志的数据包以示传达确认信息。
3. 最后，发送端再回传一个ACK标志的数据包，代表握手结束。
4. 若在握手过程中某个阶段莫名中断，TCP协议会再次以相同的顺序发送相同的数据包。

![img](https://upload-images.jianshu.io/upload_images/3334674-1f50efb3c78fc82e.png?imageMogr2/auto-orient/strip|imageView2/2/w/625/format/webp)

#### 1.3.3 DNS —— 解析域名

DNS服务是和HTTP协议一样位于应用层的协议。它提供域名到IP地址之间的解析服务。他提供域名到IP地址之间的解析服务。或者逆向从IP地址反查域名

![img](https://upload-images.jianshu.io/upload_images/3334674-6984a21fa991d1a7.png?imageMogr2/auto-orient/strip|imageView2/2/w/629/format/webp)

#### 1.4 HTTP 和各种协议的关系

![](https://theoshen-blog.oss-cn-chengdu.aliyuncs.com/blog-img/20210308162646.webp)