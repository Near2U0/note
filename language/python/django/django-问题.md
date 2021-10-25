# django.db.utils.OperationalError: (2013, ‘Lost connection to MySQL server during query‘)

报错如下

![在这里插入图片描述](../../..\images\python\20181113102823243.png)

一、数据库服务器没有开（2018-11-13遇到）
解决办法 用终端命令链接服务器试试，如报错误
mysql: [Warning] Using a password on the command line interface can be insecure. Welcome to the MySQ
说明数据库服务器没开，联系dba打开即可
二、查询速度效率过快，导致与mysql 链接数过多(2021-08-17)
在调用前调用django.db.close_old_connections关闭无效的链接。

```text
from django.db import close_old_connections
close_old_connections()
list(User.objects.filter(id=1))
[<User: admin>]
```

三、修改数据库配置文件，最大链接数
这个18年dba操作过。当时年轻不懂，现在懂了