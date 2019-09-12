# 建立连接的三次握手

# 断开连接的四次握手

# 为什么TCP传输是可靠的？
因为每次都ack~

# 滑动窗口
滑动窗口能够流量控制，让双方发送和接收的速度一致。当接收端回复ack时，默认之前的数据包都接收到。如果窗口内的数据包都得到回复，那么就易移动窗口。

# TCP的keepalive和http的keepalive的区别

# 七层模型
Please Do Not Throw Sushi Pizza Away. 
Phiscal （硬件）, Data link（地址id）, Network(地址), Transport（tcp）, Session(双方), Presentation (格式, 加密解密), Application (http/smtp).

Application：小红用http协议写了一封信
Presentation：格式是jpg，stream，json，并且进行了加密
Session：小红跟对方创建了会话
Transport：小红打算用火车来邮寄
Network：小红写上了对方的地址
Data link：小红写上了对方的邮编
Phiscal：小红把邮件送上了火车，加入了整个硬件链路

# Socket交互的基本流程
Socket指向一个file descriptors
Server创建一个socket，这个socket绑定到一个指定的端口号；
客户端知道server的ip和端口号，于是发起连接到server端；因为客户端也需要一个端口号来identity自己，于是也随机绑定了一个端口号；
于是server和client端就可以互相发送数据了
