

dpdk基于UIO绕过了传统的协议栈，避免了传统协议栈的如下问题：

1. 频繁的中断
2. 内核和用户态之间的切换及内存拷贝
3. 缓存失效



dpdk只是实现在二层，像协议栈的ARP，IP，tcp/udp协议，以及一些用户层的协议，它并没有实现，所以是比较粗糙的，如果要更高层的业务使用，还需要用户态的传输协议支持。不建议直接使用DPDK。

目前生态完善，社区强大(一线大厂支持)的应用层开发项目是[http://FD.io](https://link.zhihu.com/?target=http%3A//FD.io)(The Fast Data Project)，有思科开源支持的VPP，比较完善的协议支持，ARP、VLAN、Multipath、IPv4/v6、MPLS等。用户态传输协议UDP/TCP有TLDK。从项目定位到社区支持力度算比较靠谱的框架。





转自：https://zhuanlan.zhihu.com/p/342016978

