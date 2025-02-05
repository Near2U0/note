[TOC]





![img](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/seven-layer.png?raw=true?raw=true)


通信过程中，每层协议都要加上一个数据首部（header），称为封装（Encapsulation），如下图所示



![img](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/header-layer.png?raw=true)



# IP报文头



![image-20181017200226974](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/ip-layer.png?raw=true)

![img](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/ip-layer2.png?raw=true)



Fragment ID :段标识，如果一个数据很大的时候，会分段发送，所以段标识相同的报文是同一个报文

MF:more Fragment:是否分段

DF: Don't Frament 不允许分片

Frament Offset:如果一个报文分成了3段发送，此时3个段就会有一个offset，用来标识3段的排序，如果偏移为0，则为第一个段，如果偏移为100，则假设为第2个段，依次类推

TTL : 生存时间，每经过一个路由，减一

Protocol：协议（字段指出此数据报所携带上层数据使用的TCP协议还是UDP协议，以便对等层接收到数据报交给上层相应的协议（TCP或者UDP协议）进行处理；)

常用网际协议编号：

![img](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/ip-procol.png?raw=true)



Header CheckSum: 首部校验和，占用16位二进制数，用于协议头数据有效性的校验，可以保证IP报头区在传输时的正确性和完整性。头部检验和字段是根据IP协议头计算出的检验和，它不对头部后面的数据进行计算。

# TCP报文头

![image-20181017200410479](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/tcp-layer.png?raw=true)



![img](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/tcp-layer2.png?raw=true)



* 源端口和目的端口：各占2个字节，分别写入源端口和目的端口；
* 序号：占4个字节，TCP连接中传送的字节流中的每个字节都按顺序编号。例如，一段报文的序号字段值是 301 ，而携带的数据共有100字段，显然下一个报文段（如果还有的话）的数据序号应该从401开始；
* 确认号：占4个字节，是期望收到对方下一个报文的第一个数据字节的序号。例如，B收到了A发送过来的报文，其序列号字段是501，而数据长度是200字节，这表明B正确的收到了A发送的到序号700为止的数据。因此，B期望收到A的下一个数据序号是701，于是B在发送给A的确认报文段中把确认号置为701；
* 数据偏移，占4位，它指出TCP报文的数据距离TCP报文段的起始处有多远；
* 保留，占6位，保留今后使用，但目前应都位0；
* 紧急URG，当URG=1，表明紧急指针字段有效。告诉系统此报文段中有紧急数据；

* 确认ACK，仅当ACK=1时，确认号字段才有效。TCP规定，在连接建立后所有报文的传输都必须把ACK置1；
* 推送PSH，当两个应用进程进行交互式通信时，有时在一端的应用进程希望在键入一个命令后立即就能收到对方的响应，这时候就将PSH=1；
* 复位RST，当RST=1，表明TCP连接中出现严重差错，必须释放连接，然后再重新建立连接；
* 同步SYN，在连接建立时用来同步序号。当SYN=1，ACK=0，表明是连接请求报文，若同意连接，则响应报文中应该使SYN=1，ACK=1；
* 终止FIN，用来释放连接。当FIN=1，表明此报文的发送方的数据已经发送完毕，并且要求释放；
* 窗口，占2字节，指的是通知接收方，发送本报文你需要有多大的空间来接受；
* 检验和，占2字节，校验首部和数据这两部分；
* 紧急指针，占2字节，指出本报文段中的紧急数据的字节数；
* 选项，长度可变，定义一些其他的可选的参数。





参考：https://www.cnblogs.com/lifan3a/articles/6649970.html

https://blog.csdn.net/qzcsu/article/details/72861891



## tcp三次握手



![image-20181018194913463](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/tcp-thread.png?raw=true)



1. TCP服务器进程先创建传输控制块TCB，时刻准备接受客户进程的连接请求，此时服务器就进入了LISTEN（监听）状态；
2. TCP客户进程也是先创建传输控制块TCB，然后向服务器发出连接请求报文，这是报文首部中的同部位SYN=1，同时选择一个初始序列号 seq=x ，此时，TCP客户端进程进入了 SYN-SENT（同步已发送状态）状态。TCP规定，SYN报文段（SYN=1的报文段）不能携带数据，但需要消耗掉一个序号。
3. TCP服务器收到请求报文后，如果同意连接，则发出确认报文。确认报文中应该 ACK=1，SYN=1，确认号是ack=x+1，同时也要为自己初始化一个序列号 seq=y，此时，TCP服务器进程进入了SYN-RCVD（同步收到）状态。这个报文也不能携带数据，但是同样要消耗一个序号。
4. TCP客户进程收到确认后，还要向服务器给出确认。确认报文的ACK=1，ack=y+1，自己的序列号seq=x+1，此时，TCP连接建立，客户端进入ESTABLISHED（已建立连接）状态。TCP规定，ACK报文段可以携带数据，但是如果不携带数据则不消耗序号。
5. 当服务器收到客户端的确认后也进入ESTABLISHED状态，此后双方就可以开始通信了。 





## tcp四次断开



![image-20181018195743160](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/tcp-four.png?raw=true)

>  为什么客户端最后还要等待2MSL？

MSL（Maximum Segment Lifetime），TCP允许不同的实现可以设置不同的MSL值。

第一，保证客户端发送的最后一个ACK报文能够到达服务器，因为这个ACK报文可能丢失，站在服务器的角度看来，我已经发送了FIN+ACK报文请求断开了，客户端还没有给我回应，应该是我发送的请求断开报文它没有收到，于是服务器又会重新发送一次，而客户端就能在这个2MSL时间段内收到这个重传的报文，接着给出回应报文，并且会重启2MSL计时器。

第二，防止类似与“三次握手”中提到了的“已经失效的连接请求报文段”出现在本连接中。客户端发送完最后一个确认报文后，在这个2MSL时间中，就可以使本连接持续的时间内所产生的所有报文段都从网络中消失。这样新的连接中不会出现旧连接的请求报文。

为什么建立连接是三次握手，关闭连接确是四次挥手呢？

建立连接的时候， 服务器在LISTEN状态下，收到建立连接请求的SYN报文后，把ACK和SYN放在一个报文里发送给客户端。 
而关闭连接时，服务器收到对方的FIN报文时，仅仅表示对方不再发送数据了但是还能接收数据，而自己也未必全部数据都发送给对方了，所以己方可以立即关闭，也可以发送一些数据给对方后，再发送FIN报文给对方来表示同意现在关闭连接，因此，己方ACK和FIN一般都会分开发送，从而导致多了一次。



## tcp的有限状态机

| TCP includes 11 states, they are: |
| --------------------------------- |
| LISTEN                            |
| SYN_SENT                          |
| SYN_RECV                          |
| ESTABLISHED                       |
| FIN_WAIT1                         |
| CLOSE_WAIT                        |
| FIN_WAIT2                         |
| LAST_ACK                          |
| TIME_WAIT                         |
| CLOSED                            |
| CLOSING                           |



### 状态视图

* 服务器端

![img](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/tcp-status.png?raw=true)

* 客户端





![img](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/tcp-status2.png?raw=true)

* 总的状态图



![img](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/tcpfsm.png?raw=true)



# 防火墙



工作在主机或者是网络的**边缘**，对于进出的数据报文能够根据**事先定义好的规则**，做出相应的处理的组件，称之为网络防火墙或者主机防火墙.



## 四表五链

规则：匹配标准

​	IP：SIP， DIP

​	TCP：SPort，DPort  	 SYN=1,FIN-0,RST=0,ACK=0(TCP三次握手中的第一次)；SYN=1,ACK=1,FIN=0,RST=0(TCP三次握手中的第2次);ACK=1,FIN=0,SYN=0,RST=0(establish 以建立连接)

​	UDP：SPort，DPort

​	ICMP: icmp-type



数据报文过滤

Linux2.2

​	ipchain(用户空间命令)/firewall（内核空间命令）

Linux2.4

​	iptables(用户空间命令)/netfilter（内核空间命令）



![image-20181018234248087](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/sys-user.png?raw=true)



在用户空间中有（iptables,或者是ipchain)这样一个工具，我们可以使用这样的工具编写规则，然后将规则刷新到内核空间中，netfilter会根据规则达到防火墙的目的



hook function:5个钩子函数

* prerouting 刚到达本机

* input：进入本机（1）
* output:从本机出去（2）
* forward从本机转发（3）
* postrouting 从本机发送出去



![image-20181020090657434](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/hook.png?raw=true)



把访问频繁的请求放在上面，避免过多的检查（如web服务）

通常在一个数据报文送达到网卡之后，网卡会将数据报文送达到路由表进行路由决策，但是在网卡上也有一个**钩子函数**，用于路由之前的修改数据报文的内容，同样在数据报文通过网卡发送出去之前，我们也是可以修改数据报文的内容，这里也是对应一个钩子函数。

![image-20181020091741755](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/hook2.png?raw=true)



在网卡的出口和入口位置，我们都可以对数据报文的内容进行修改，如：实现地址转换的功能



规则链：每一个规则函数上提供的规则，将组成一个链，所以就有了下面这5个规则链

​	PREROUTING

​	INPUT

​	FORWARD

​	OUTPUT

​	POSTROUTING

功能：

filter（过滤）：这里每个链都对应一个列，每个链内部又有多行，所以组成了 **表**

​	INPUT 能不能到达本机

​	OUTPUT 能不能从本机发出

​	FORWARD 能不能经由本机转发

nat(地址转换)：这里每个链都对应一个列，每个链内部又有多行，所以组成了 **表**

​	PREROUTING

​	OUTPUT

​	POSTROUTING

mangle(修改报文首部):表

​	修改ttl值等

​	PREROUTING

​	INPUT

​	OUTPUT

​	FORWARD

​	POSTROUTING

raw(原始格式)：表

​	不做任何修改

​	PREROUTING

​	OUTPUT



![image-20181020133158472](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/chains-type.png?raw=true)



这就是所谓的 **四表五链**



自定义链

可以使用自定义链，但只在被调用时才能发挥作用，而且如果没有被任何一条规则匹配到，还应该有返回机制,用户可以删除自定义的空链，默认链不能被删除



# 规则



每一条规则都有两个内置的计数器

​	被匹配到的报文个数

​	被匹配的报文体积之和



规则：是由 **匹配标准** 和 **处理动作** 组成

iptable [-t TABLE]  COMMAND  CHAIN [num] 匹配条件  -j 处理办法

如果-t table没有写，默认是 -t filter 表





## 匹配条件

### 通用匹配

| 匹配标准                | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| -s , —src               | 指定源地址                                                   |
| -d , —dst               | 指定目标地址                                                 |
| -p {tcp \| udp \| icmp} | 指定协议                                                     |
| -i  interface           | 指定数据报文流入的接口（从哪个网卡进入的），在prerouting，input, forward中使用才有意义 |
| -o interface            | 指定数据报文流出的接口（从哪个网卡出去的），在postrouting,output,forward中使用才有意义 |

### 扩展匹配（调用netfilter的扩展模块）



#### 隐含扩展

  不用特别指定由哪个模块进行扩展 ，因为此时使用了 -p(tcp|udp|icmp),比如如果我们使用了tcp之后，就可以使用tcp这个特定协议的扩展，可以指定源端口，目标端口进行匹配

  ```
  -p tcp
  	--sport PORT [-PROT2]  ：源端口
  	--dport PORT  [-PROT2]  ：目标端口
  可以使用连续的端口，如 ： --sprot 80-110
  	--tcp-flags mask comp : 通过tcp的标志位匹配
  		--tcp-flags SYN,FIN,ACK,RST  SYN,ACK  只检查mask指定的标志位(SYN,FIN,ACK,RST)，comp表示此列表中的位必须为1(SYN,ACK都为1)，comp中出现的，但是mask中没有出现的，必须为0，如：FIN和RST必须为0
  		--tcp-flags SYN,FIN,ACK,RST  SYN ：表示只有SYN为1，其他都为0，表示三次握手中的第一次，简写为--syn
  		--syn :匹配三次握手中的第一次
  		
  #放行172.16网络的 访问172.16.100.7 主机的ssh服务
  ##进入放行
  iptables -t filter -A INPUT -s 172.16.0.0/16 -d 172.16.100.7 -p tcp --dport 22 -j ACCEPT
  ##出去放行
  iptables -t filter -A OUTPUT -s 172.16.100.7 -d 172.16/16 -p tcp --sport 22 -j ACCEPT
  
  
  
  -p icmp
  		--icmp-type :指定icmp协议报文的类型
  			0:echo-reply ping响应报文
  			8:echo-request ping请求报文
  			
  #放行本机ping
  iptables -A OUTPUT -s 172.16.110.7 -p icmp --icmp-type 8 -j ACCEPT
  iptables -A INPUT -d 172.16.110.7 -p icmp --icmp-type 0 -j ACCEPT
  
  
  -p udp
  	--sport
  	--dport
  	
  #DNS服务器提供服务:(从内网主机到DNS服务器，从DNS服务器到根服务器)，所以tcp需要些四条规则，udp需要些四条规则，总共8条规则
  
  
  ```

  

  DNS服务的请求场景

  * 客户端请求DNS服务器
  * DNS服务器请求根服务器

  

  ![image-20181020175106434](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/dns-open.png?raw=true)

  



#### 显式扩展

必须指明由哪个模块进行的扩展，在iptable中使用-m选项可完成此功能



##### state:状态扩展

   结合ip_conntrack追踪会话的状态，

​	NEW:新连接的请求

​	ESTABLISHED:以建立的连接，对于新请求的相应也是已建立的连接

​	INVALID : 非法连接请求（如：SYN=1, FIN=1)

​	RELATED:相关联的，如在ftp服务中，建立连接使用的是21号端口，但是后面的数据请求使用的是一个随机的端口，此时就用到了**相关联的**这种状态，表示跟此前的状态有关联

​		-m state - -state NEW,ESTABLISHED -j ACCEPT


```
#接收来自ssh的请求(如果是状态追踪，那么请求进来是有new，establish，但是出去只能是establish)
iptables -A INPUT -d 172.16.100.7 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -A OUTPUT -s 172.16.100.7 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT



#出去的报文都是ESTABLISHED，所以出去我们只需要写一条规则就行了，并把它放在第一条，这样实现最先匹配
iptables -I OUTPUT -s 172.16.100.7 -m state --state ESTABLISHED -j ACCEPT


#开放ftp服务(首先需要装载ip_conntrack_ftp , ip_nat_ftp模块)
iptables -A INPUT -d 172.16.100.7 -p tcp -m state --state RELATED,ESTABLISHED -j ACCEPT
##修改出去的第一条
iptables -R OUTPUT 1 -s 172.16.100.7 -p tcp -m state --state ESTABLISHED,RELATED -j ACCEPT

```

​	

##### multiport：离散多端口匹配扩展

```
-m multiport
	--source-ports 
	--destination-ports
	--ports		不分区源和目标
	
iptables -I INPUT 2 -d 172.16.100.7 -p tcp -m multiport --destination-ports 21,22,80 -m state --state NEW -j ACCEPT

```



##### iprange,ip范围扩展

```
-m iprange 
    --src-range ip-ip
    --dst-range ip-ip

#当前的ssh服务，仅放行给172.16.100.3 - 172.16.100.100 访问
iptables -A INPUT -p tcp --dport 22 -m iprange --src-range 172.16.100.3-172.16.100.100 -m state --state NEW,ESTABLISHED -j ACCEPT

```



##### connlimit,连接数限定

可以限定某一个IP，最多发起多少个连接请求

```
! --connlimit-above n  #连接的上限,注意 ! 的作用
--connlimit-above 2 -j ACCEPT #表示达到2个的上限就放行，但是加上的!就表示没有达到2个就放行，所以一般需要加上！

#表示低于2个的连接就放行(没有达到)
iptables -A INPUT -d 172.16.100.7 -p tcp --dport 80 -m connlimit ! --connlimit-above 2 -j ACCEPT


```



##### limit

速率限定，每单位时间的请求数

```
每单位时间接入的请求数，以及并发接入的请求数
--limit rate  #每秒，或者每分钟，或者每小时可以进入多少连接 default 3/hour 
--limit-burst  #并发接入的连接数

#ssh服务，允许每分钟只接入3个请求
iptables -I INPUT -d 172.16.100.7 -p tcp --dport 22 -m limit ---limit 3/minute --limit-burst 3 -j ACCEPT


#限制每分钟5个ping请求
iptables -A INPUT -d 172.16.100.7 -p icmp --icmp-type 8 -m limit --limit 5/minute -j ACCEPT



```

##### string

字符串匹配过滤

```
--algo {bm|kmp}   #指定一个算法
--string "STRING"  #需要匹配的字符串



#包含 h7n9的字符串都拒绝(注意是OUTPUT的请求，所以是出去的报文有字符串过滤的情况)
iptables -I OUTPUT -s 172.16.100.7 -m string --algo kmp --string "h7n9" -j REJECT

```



​	



* 保存规则

  service iptables save	 会保存文件到 /etc/sysconfig/iptables

  或者

  iptables-save > /etc/sysconfig/iptables.2018

  iptables-restore </etc/sysconfig/iptables.2018  #使生效


  ```
  iptables重启的时候，会读取下面的文件到内存中
  [root@localhost ~]# cat /etc/sysconfig/iptables
  # Firewall configuration written by system-config-firewall
  # Manual customization of this file is not recommended.
  *filter
  :INPUT ACCEPT [0:0]
  :FORWARD ACCEPT [0:0]
  :OUTPUT ACCEPT [0:0]
  -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
  -A INPUT -p icmp -j ACCEPT
  -A INPUT -i lo -j ACCEPT
  -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
  -A INPUT -j REJECT --reject-with icmp-host-prohibited
  -A FORWARD -j REJECT --reject-with icmp-host-prohibited
  COMMIT
  [root@localhost ~]# 
  ```

  

  

  





连接追踪

在 /proc/net/ip_conntrack 文件中保存了当前主机和其他外面的主机的连接信息

![image-20181020180430565](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/ip_conntrack.png?raw=true)

可以修改这个值

```
sysctl -w net.ipv4.ip_conntrack_max=65536
```



 











## 处理动作

-j target

| TARGET     |                                                              |
| ---------- | ------------------------------------------------------------ |
| ACCEPT     | 接收（放行）                                                 |
| DROP       | 悄悄拒绝                                                     |
| REJECT     | 明确拒绝，测试目的建议使用，一般不建议使用                   |
| DNAT       | 目标地址转换                                                 |
| SNAT       | 源地址转换                                                   |
| REDIRECT   | 端口重定向                                                   |
| MASQUERADE | mas querade 地址伪装，在nat表中postrouting链中实现**源地址转换**的 |
| LOG        | 记录日志                                                     |
| MARK       | 给一个报文打一个标记                                         |



```
#来自172.16这个网段的地址，都拒绝掉(drop)
iptable  -t filter -A INPUT -s 172.16.0.0/16 -j DROP


#来自172.16这个网段的地址，且访问 172.16.100.7 的主机都拒绝掉(drop)
iptable  -t filter -A INPUT -s 172.16.0.0/16 -d 172.16.100.7 -j DROP

```



> LOG 记录日志

```
--log-level 
--log-prefix prefix
--log-tcp-sequence
--log-tcp-options
--log-ip-options
--log-uid

iptables -I INPUT -d 172.16.100.7 -p icmp --icmp-type 8 -j LOG --log-prefix "firewall log for icmp--"

tail -f /var/log/messages	#查看记录的日志
```



​	

## 命令command



### 管理规则

-A :附加一条规则，在链的尾部追加，

-I chain [num] : 插入一条规则，指定插入到chain链上的第num条的位置 ，省略num则为第一条

-D chain [num] : 删除指定链中的第num条规则 :iptables -D INPUT 1  表示删除第一条规则

-R chain [num]: 替换指定的规则



### 管理链：

-F [chain]：flush，清空指定规则链，如果省略chain，则删除对应表中的所有链

-P chain [ACCETP|DROP] :指定链的默认策略

-N :自定义一个新的空链

-S：删除一个自定义的空链，如果是非空，先使用-F清空

-Z:清空计数器（置零）

-E :重命名一条自定义链

```
#将链的默认策略改为drop，最好事先写好能够ssh的规则，不然会将ssh也挡在外面，这样就不能远程了
iptables -P INPUT DROP 
iptables -P OUTPUT DROP 
iptables -P FORWARD DROP 

#放行web服务
iptables -I INPUT  -d 172.16.100.7 -p tcp --dport 80 -j ACCEPT
iptables -I OUTPUT -s 172.16.100.7 -p tcp --sport 80 -j ACCEPT
#查看
[root@localhost ~]# iptables -L -n
Chain INPUT (policy DROP)
target     prot opt source               destination         
ACCEPT     tcp  --  0.0.0.0/0            172.16.100.7        tcp dpt:80 
Chain OUTPUT (policy DROP)
target     prot opt source               destination         
ACCEPT     tcp  --  172.16.100.7         0.0.0.0/0           tcp spt:80 


#放行本地的ping
iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -i lo -j ACCEPT
iptables -A OUTPUT -s 127.0.0.1 -d 127.0.0.1 -o lo -j ACCEPT


#放行icmp


```



#### 自定义链

```
#创建一个新的链
iptables -N clean_in

iptables -A clean_in -d 255.255.255.255 -p icmp -j DROP
iptables -A clean_in -d 172.16.255.255 -p icmp -j DROP
iptables -A clean_in -p tcp ! --syn -m state --state NEW -j DROP
#返回主链
iptables -A clean_in -d 172.16.100.7 -j RETURN

#被主链调用
iptables -I INPUT -j clean_in



```

由下图，我们可以看到，首先在主链中，进入了chen_in链的调用，同时可以看到clean_in链是被引用了一次（也就是主链引用）

![image-20181021221047366](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/udc.png?raw=true)



### 查看链

-L：擦汗指定表中的规则（会将IP反解为主机名，将端口反解为协议名称）

​	-n : 以数字格式显示主机地址和端口号

​	-v : 显示详细信息（计数器信息：接收的报文数量，字节总大小）

​	--line-numbers :显示规则号码

```
iptables -t filter -L -n 
#简写,默认是filter表
iptables -L -n



#以IP和端口号显示
[root@localhost ~]# iptables -L -n 
Chain INPUT (policy ACCEPT 默认的策略是 ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 


#显示计数器详情：这里可以看到通过每条规则的包的数量和字节数
[root@localhost ~]# iptables -L -n -v
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  306 30874 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
  
#显示规则的行号        
[root@localhost ~]# iptables -L -n --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
2    ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
3    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
4    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
5    REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

#查看指定的链
root@bdsoc-pc:~# iptables --line-number -L INPUT 
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    ACCEPT     all  --  anywhere             172.16.100.7        
root@bdsoc-pc:~#
```



iptables不是服务，但是有服务脚本，服务脚本的主要作用在于管理保存的规则，装载及移除iptables/netfilter相关的内核模块：iptables_nat, iptables_filter,iptables_mangle, iptabels_raw, ip_nat, ip_conntrack



# iptables抵御dos攻击



![image-20181021222000293](https://github.com/chenyansong1/note/blob/master/images/linux/iptables/dos.png?raw=true)





# icmp code 对照表



| TYPE | CODE | Description                                                  | Query | Error |
| ---- | ---- | ------------------------------------------------------------ | ----- | ----- |
| 0    | 0    | Echo Reply——回显应答（Ping应答）                             | x     |       |
| 3    | 0    | Network Unreachable——网络不可达                              |       | x     |
| 3    | 1    | Host Unreachable——主机不可达                                 |       | x     |
| 3    | 2    | Protocol Unreachable——协议不可达                             |       | x     |
| 3    | 3    | Port Unreachable——端口不可达                                 |       | x     |
| 3    | 4    | Fragmentation needed but no frag. bit set——需要进行分片但设置不分片比特 |       | x     |
| 3    | 5    | Source routing failed——源站选路失败                          |       | x     |
| 3    | 6    | Destination network unknown——目的网络未知                    |       | x     |
| 3    | 7    | Destination host unknown——目的主机未知                       |       | x     |
| 3    | 8    | Source host isolated (obsolete)——源主机被隔离（作废不用）    |       | x     |
| 3    | 9    | Destination network administratively prohibited——目的网络被强制禁止 |       | x     |
| 3    | 10   | Destination host administratively prohibited——目的主机被强制禁止 |       | x     |
| 3    | 11   | Network unreachable for TOS——由于服务类型TOS，网络不可达     |       | x     |
| 3    | 12   | Host unreachable for TOS——由于服务类型TOS，主机不可达        |       | x     |
| 3    | 13   | Communication administratively prohibited by filtering——由于过滤，通信被强制禁止 |       | x     |
| 3    | 14   | Host precedence violation——主机越权                          |       | x     |
| 3    | 15   | Precedence cutoff in effect——优先中止生效                    |       | x     |
| 4    | 0    | Source quench——源端被关闭（基本流控制）                      |       |       |
| 5    | 0    | Redirect for network——对网络重定向                           |       |       |
| 5    | 1    | Redirect for host——对主机重定向                              |       |       |
| 5    | 2    | Redirect for TOS and network——对服务类型和网络重定向         |       |       |
| 5    | 3    | Redirect for TOS and host——对服务类型和主机重定向            |       |       |
| 8    | 0    | Echo request——回显请求（Ping请求）                           | x     |       |
| 9    | 0    | Router advertisement——路由器通告                             |       |       |
| 10   | 0    | Route solicitation——路由器请求                               |       |       |
| 11   | 0    | TTL equals 0 during transit——传输期间生存时间为0             |       | x     |
| 11   | 1    | TTL equals 0 during reassembly——在数据报组装期间生存时间为0  |       | x     |
| 12   | 0    | IP header bad (catchall error)——坏的IP首部（包括各种差错）   |       | x     |
| 12   | 1    | Required options missing——缺少必需的选项                     |       | x     |
| 13   | 0    | Timestamp request (obsolete)——时间戳请求（作废不用）         | x     |       |
| 14   |      | Timestamp reply (obsolete)——时间戳应答（作废不用）           | x     |       |
| 15   | 0    | Information request (obsolete)——信息请求（作废不用）         | x     |       |
| 16   | 0    | Information reply (obsolete)——信息应答（作废不用）           | x     |       |
| 17   | 0    | Address mask request——地址掩码请求                           | x     |       |
| 18   | 0    | Address mask reply——地址掩码应答                             |       |       |

http://www.cnitblog.com/yang55xiaoguang/articles/59581.html





# NAT 

Network Address Translation



* DNAT 目标地址转换
* SNAT 源地址转换



```
-j SNAT

	--to-source #指定源地址  --to-source 123.2.3.2-123.2.3.5 指定多个地址
-j MASQUERADE #可以动态的选择外网地址，这样不用我们指定外网地址
	

-j DNAT
	--to-destination IP[:port]

#原地址转换，使内网能够上网
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j SNAT --to-source 123.2.3.2


#目录地址转换
##场景：假设我们只有一个公网IP，但是我们内网中有web服务器和ftp服务器，我们希望外网访问80的时候，我们将目标地址转换成web服务器，外网访问21的时候，我们将目标地址转换成ftp服务器，这就是所谓的 端口映射
iptables -t nat -A PREROUTING -d 123.2.3.2 -p tcp --dport 80 -j DNAT --to-destination 192.168.10.22

iptables -t nat -A PREROUTING -d 123.2.3.2 -p tcp --dport 21 -j DNAT --to-destination 192.168.10.33

#服务器做了返回的地址转换


#查看定义的映射
iptables -t nat -vnL PREROUTING --line-number


#改变目标地址的同时改变端口
iptables -t nat -A PREROUTING -d 111.11.11.1 -p tcp --dport 80 -j DNAT --to-destination 192.168.10.33:8080


```





# layer7



iptables并不能实现应用层协议的过滤，此时就有了layer7



需要给内核打补丁

需要给iptables打补丁

https://blog.csdn.net/u014721096/article/details/78629523



​	