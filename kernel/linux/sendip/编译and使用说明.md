[toc]



# 编译

上传的..\kernel\linux\sendip\sendip-2.5.2021.tar.gz  以及解决了编译的错误问题

源码地址：http://www.earth.li/projectpurple/progs/sendip.html

编译错误解决：https://blog.csdn.net/figo1986/article/details/7336131



# usage

reference：https://www.open-open.com/lib/view/open1429781017854.html

```shell
Sendip Examples

#UDP Packet

sendip -p ipv4 -is 192.168.1.21 -p udp -us 5070 -ud 5060 -d "UDP Test" -v 192.168.1.21

#ICMP Packet

sendip 192.168.2.12 -p icmp -is 192.168.2.22

#TCP Packet

sendip 192.168.2.12 -p tcp -ts 2 -td 80 -tn -is 192.168.2.22

```



ref:https://www.cnblogs.com/starspace/archive/2009/01/15/1376638.html





