4次挥手和timewait

1. 为什么连接的时候是三次握手，关闭的时候却是四次握手？

	因为当Server端收到Client端的SYN连接请求报文后，可以直接发送SYN+ACK报文。

	其中**ACK报文是用来应答**的，**SYN报文是用来同步**的。

	但是关闭连接时，当Server端收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉Client端，"你发的FIN报文我收到了"。只有等到我Server端所有的报文都发送完了，我才能发送FIN报文，因此不能一起发送。故需要四步握手。

2. #### TIME_WAIT

	客户端收到服务的释放连接的请求后，**不是立马进入CLOSE状态**，而是还要再等待2MSL。理由是：

	- 确保最后一个确认报文能够到达。如果没能到达，服务端就会会重发FIN请求释放连接。等待一段时间没有收到重发就说明服务的已经CLOSE了。如果有重发，则客户端再发送一次LAST ack信号
	- 等待一段时间是为了让本连接持续时间内所产生的所有报文都从网络中消失，使得下一个新的连接不会出现旧的连接请求报文。

<https://wizardforcel.gitbooks.io/network-basic/7.html>

<https://blog.csdn.net/dangzhangjing97/article/details/81008836>

