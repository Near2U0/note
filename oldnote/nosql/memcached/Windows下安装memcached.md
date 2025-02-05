---
title: Windows下安装memcached
categories: memcached   
toc: true  
tags: [memcached]
---



# 1.windows下安装
 
只需要将<font color=red>memcached.exe</font> 放在相应的目录下，然后在cmd下，进入目录，执行：
 
## 1.1.查看帮助文档
```
D:\installed_soft\memcached>memcached.exe -h
 
D:\installed_soft\memcached>memcached.exe -h
memcached 1.4.4-14-g9c660c0
-p <num>      TCP port number to listen on (default: 11211)                                #监听的端口
-U <num>      UDP port number to listen on (default: 11211, 0 is off)
-s <file>     UNIX socket path to listen on (disables network support)
-a <mask>     access mask for UNIX socket, in octal (default: 0700)
-l <ip_addr>  interface to listen on (default: INADDR_ANY, all addresses)
-s <file>     unix socket path to listen on (disables network support)
-a <mask>     access mask for unix socket, in octal (default 0700)
-l <ip_addr>  interface to listen on, default is INADDR_ANY
-d start          tell memcached to start                                                                #启动
-d restart        tell running memcached to do a graceful restart
-d stop|shutdown  tell running memcached to shutdown
-d install        install memcached service
-d uninstall      uninstall memcached service
-r            maximize core file limit
-u <username> assume identity of <username> (only when run as root)                        #指定用户
-m <num>      max memory to use for items in megabytes (default: 64 MB)                        #指定内存
-M            return error on memory exhausted (rather than removing items)
-c <num>      max simultaneous connections (default: 1024)
-k            lock down all paged memory.  Note that there is a
              limit on how much memory you may lock.  Trying to
              allocate more than that would fail, so be sure you
              set the limit correctly for the user you started
              the daemon with (not for -u <username> user;
              under sh this is done with 'ulimit -S -l NUM_KB').
-v            verbose (print errors/warnings while in event loop)
-vv           very verbose (also print client commands/reponses)
-vvv          extremely verbose (also print internal state transitions)                          #指定打印详细信息
-h            print this help and exit
-i            print memcached and libevent license
-P <file>     save PID in <file>, only used with -d option
-f <factor>   chunk size growth factor (default: 1.25)                                                #指定增长因子
-n <bytes>    minimum space allocated for key+value+flags (default: 48)
-L            Try to use large memory pages (if available). Increasing
              the memory page size could reduce the number of TLB misses
              and improve the performance. In order to get large pages
              from the OS, memcached will allocate the total item-cache
              in one large chunk.
-D <char>     Use <char> as the delimiter between key prefixes and IDs.
              This is used for per-prefix stats reporting. The default is
              ":" (colon). If this option is specified, stats collection
              is turned on automatically; if not, then it may be turned on
              by sending the "stats detail on" command to the server.
-t <num>      number of threads to use (default: 4)
-R            Maximum number of requests per event, limits the number of
              requests process for a given connection to prevent
              starvation (default: 20)
-C            Disable use of CAS
-b            Set the backlog queue limit (default: 1024)
-B            Binding protocol - one of ascii, binary, or auto (default)
-I            Override the size of each slab page. Adjusts max item size
              (default: 1mb, min: 1k, max: 128m)
```
 
 
 
 
## 1.2.启动
```
D:\installed_soft\memcached>memcached.exe -m 64 -p 11211 -vv                           #指定启动时监听的端口，开64M内存，打印详细信息（vvv )
 
```
 
 
## 1.3.测试
```
#另外开启一个cmd窗口，使用Telnet去连接memcached：
C:\Users\Administrator>telnet 127.0.0.1 11211
 
#添加一条数据
add new 0 0 8
zhangsan                                        #值
STORED                                        #表示添加成功
```







