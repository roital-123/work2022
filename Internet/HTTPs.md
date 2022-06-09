# HTTPs如何通过SSL/TSL保证了传输的安全性

- 重点理解证书？



# TCP状态时序图

- TCP中各个状态转换图？

  <img src="/Users/apple/Documents/work2022/C++ knowledge/picture/TCP通信时序图.png" alt="TCP通信时序图" style="zoom:67%;" />

  ​	 	**主动建立连接方**：**CLOSED** ---- 发送SYN后 ---- **SYN_SEND** ---- 收到对端ACK和SYN ---- **ESTABLISHED**；

  ​		**被动接收连接方**：**CLOSED** ---- **LISTEN** ---- 收到对端发送的SYN，发送SYN和ACK ---- **SYN_RECV** ----收到对端ACK ---- **ESTABLISHED**；

  ​		**主动断开连接方**：**ESTABLISHED** ---- 发送FIN后 ---- **FIN_WAIT1** ---- 收到对端发送的ACK后 ---- **FIN_WAIT2**（此时处于半关闭状态）---- 收到对端发送的FIN后并发送ACK ---- **TIME_WAIT** ---- 等待2MSL ---- **CLOSED**；

  ​		**被动接收断开连接方**：**ESTABLISHED** ---- 接收到对端SYN并发送ACK后 ---- **CLOSED_WAIT** ---- 发送SYN后 ---- **LAST_ACK** ---- 接收到对端ACK后 ---- **CLOSED**；

- time_wait状态产生的原因？危害？如何避免？

  - [原因详解](https://blog.csdn.net/huangyimo/article/details/81505558)

  - **原因1）**为实现TCP这种全双工连接的可靠释放

    ​		如果主动关闭一方（视为client）发送的**ACK**（4次挥手的最后一个包）在网络中丢失，由于TCP重传机制，被动关闭的一方会重发其**FIN**，在FIN到达client之前，client必须维护这条连接的状态。具体而言就是这条TCP连接对应的资源不能被立即释放或重新分配。

    ​		如果client不进入TIME_WAIT以维护其连接状态，当server重发的FIN到达时，client方**TCP传输层**会以**RST包**响应对方，这会被对方认为有错误发生。

  - **原因2）**为使旧的数据包在网络因过期而消失

    ​		假设TCP协议中不存在TIME_WAIT状态的限制，再假设当前有一条TCP连接：**(local_ip, local_port, remote_ip,remote_port)**，因为某些原因，我们先关闭，接着很快以相同的四元组建立一条新连接。由于TCP连接由四元组唯一标识，因此，在假设的情况中，TCP协议栈是无法区分前后两条TCP连接的不同的，在它看来，这根本就是同一条连接，中间先释放再建立的过程对其来说是“感知”不到的。

    ​		这样就可能发生这样的情况：**前一条TCP连接**由local peer发送的数据到达remote peer后，会被该remot peer的TCP传输层**当做当前TCP连接**的正常数据接收并向上传递至应用层（而事实上，这些旧数据到达remote peer前，旧连接已断开且一条由相同四元组构成的新TCP连接已建立，因此，这些旧数据是不应该被向上传递至应用层的），从而引起数据错乱进而导致各种无法预知的诡异现象。

    ​		 具体而言，local peer主动调用close后，此时的TCP连接进入TIME_WAIT状态，处于该状态下的TCP连接不能立即以同样的四元组建立新连接，即**主动关闭方**占用的**local port**在TIME_WAIT期间不能再被重新分配。由于TIME_WAIT状态持续时间为2MSL，这样保证了旧TCP连接双工链路中的旧数据包均因过期（超过MSL）而消失，此后，就可以用相同的四元组建立一条新连接而不会发生前后两次连接数据错乱的情况。

  - 避免：**端口复用**

    `int opt = 1; setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));`

