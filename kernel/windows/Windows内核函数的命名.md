[toc]

# Windows内核函数的命名

函数名都按照其所在的层次或模块加上了特定的前缀

```text
Ex: 管理层， Executive的开头两个字母
Ke：核心层，Kernel开头的两个字母
Ke表示属于“内核管理科层”，Ki表示内核中的底层，与中断，异常，自陷相关
Hal: 硬件抽象层，Hardware Abstraction Layer
Ob:对象管理，Object开头
Mm：内存管理，Memory Manager
Ps:进程管理，Ps表示Process
Se：安全管理，Security
Io: I/O管理
Fs: File System
Cc: 文件缓存管理，Cc Cache
Cm: 系统配置管理，Configuration Manager
Pp: “即插即用”管理，Pp 表示PnP
Rtl：运行时程序库，Rtl is Runtime Library
```



Ke表示属于“内核管理科层”，Ki表示内核中的底层，与中断，异常，自陷相关
