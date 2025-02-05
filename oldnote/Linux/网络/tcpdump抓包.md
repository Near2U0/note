* 安装

```
tcpdump不是默认安装的，在CentOS下安装：
yum install tcpdump

在Ubuntu下安装：
apt-get install tcpdump
```



* 常见命令

```
tcpdump
-i 	//监视指定网络接口的数据包(一定要查看网卡)，默认是监视**第一个网络接口**上所有流过的数据包，-i any是监视所有的网卡


1)tcp: ip icmp arp rarp 和 tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型
(2)-i eth1 : 只抓经过接口eth1的包
(3)-t : 不显示时间戳
(4)-s 0 : 抓取数据包时默认抓取长度为68字节。加上-S 0 后可以抓到完整的数据包
(5)-c 100 : 只抓取100个数据包
(6)dst port ! 22 : 不抓取目标端口是22的数据包
(7)src net 192.168.1.0/24 : 数据包的源网络地址为192.168.1.0/24
(8)-w ./target.cap : 保存成cap文件，方便用ethereal(即wireshark)分析
(9)-n 以数字显示主机及端口：tcpdump -n


-n     Don't convert host addresses to names.  This can be used to avoid DNS lookups.
-nn    Don't convert protocol and port numbers etc. to names either.


#一般的web服务器都会有80的包
tcpdump -i eno1 -nn -n -s 3000 tcp and dst port 80
```





* 抓包

```
tcpdump -i any host 10.130.10.22 and port 9092
```



```
========================================================================
###################过滤主机###################
========================================================================
- 抓取所有经过 eth1，目的或源地址是 192.168.1.1 的网络数据
# tcpdump -i eth1 host 192.168.1.1
- 源地址
# tcpdump -i eth1 src host 192.168.1.1
- 目的地址
# tcpdump -i eth1 dst host 192.168.1.1


========================================================================
###################过滤端口###################
========================================================================
- 抓取所有经过 eth1，目的或源端口是 25 的网络数据
# tcpdump -i eth1 port 25
- 源端口
# tcpdump -i eth1 src port 25
- 目的端口
# tcpdump -i eth1 dst port 25


========================================================================
###################网络过滤###################
========================================================================
# tcpdump -i eth1 net 192.168
# tcpdump -i eth1 src net 192.168
# tcpdump -i eth1 dst net 192.168


========================================================================
###################协议过滤###################
========================================================================
# tcpdump -i eth1 arp
# tcpdump -i eth1 ip
# tcpdump -i eth1 tcp
# tcpdump -i eth1 udp
# tcpdump -i eth1 icmp


========================================================================
###################常用表达式###################
========================================================================
非 : ! or "not" (去掉双引号)
且 : && or "and"
或 : || or "or"
- 抓取所有经过 eth1，目的地址是 192.168.1.254 或 192.168.1.200 端口是 80 的 TCP 数据
# tcpdump -i eth1 '((tcp) and (port 80) and ((dst host 192.168.1.254) or (dst host
192.168.1.200)))'
- 抓取所有经过 eth1，目标 MAC 地址是 00:01:02:03:04:05 的 ICMP 数据
# tcpdump -i eth1 '((icmp) and ((ether dst host 00:01:02:03:04:05)))'
- 抓取所有经过 eth1，目的网络是 192.168，但目的主机不是 192.168.1.200 的 TCP 数据
# tcpdump -i eth1 '((tcp) and ((dst net 192.168) and (not dst host 192.168.1.200)))'



- 只抓 SYN 包
# tcpdump -i eth1 'tcp[tcpflags] = tcp-syn'
- 抓 SYN, ACK
# tcpdump -i eth1 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack != 0'

- 抓 SMTP 数据
# tcpdump -i eth1 '((port 25) and (tcp[(tcp[12]>>2):4] = 0x4d41494c))'
抓取数据区开始为"MAIL"的包，"MAIL"的十六进制为 0x4d41494c。

- 抓 HTTP GET 数据
# tcpdump -i eth1 'tcp[(tcp[12]>>2):4] = 0x47455420'
"GET "的十六进制是 47455420


- 抓 SSH 返回
# tcpdump -i eth1 'tcp[(tcp[12]>>2):4] = 0x5353482D'
"SSH-"的十六进制是 0x5353482D


# tcpdump -i eth1 '(tcp[(tcp[12]>>2):4] = 0x5353482D) and (tcp[((tcp[12]>>2)+4):2]
= 0x312E)'抓老版本的 SSH 返回信息，如"SSH-1.99.."

- 抓 DNS 请求数据
# tcpdump -i eth1 udp dst port 53



其他
----
-c 参数对于运维人员来说也比较常用，因为流量比较大的服务器，靠人工 CTRL+C 还是
抓的太多，于是可以用-c 参数指定抓多少个包。
# time tcpdump -nn -i eth0 'tcp[tcpflags] = tcp-syn' -c 10000 > /dev/null
上面的命令计算抓 10000 个 SYN 包花费多少时间，可以判断访问量大概是多少。



实时抓取端口号8000的GET包，然后写入GET.log

tcpdump -i eth0 '((port 8000) and (tcp[(tcp[12]>>2):4]=0x47455420))' -nnAl -w /tmp/GET.log


```





* 抓包保存到文件

```
tcpdump tcp -i any -t -s 0 -c 100 and host 10.130.10.22 and port 9092 -w ./target-2019011509.cap

```










