[TOC]



# MIME的由来

MIME:Multipurpose Internet Mail Extension, 多用途互联网邮件扩展



原来的邮件服务中，只能够支持传输文本，但是为了传输其他的问题，如mp3等这样的二进制文件，所有有了MIME，**将非文本的数据在传输前重新编码为文本格式，接收方能够用相反的方式将其重新还原为原来的格式，还能够调用相应的程序来打开此文件**



在http/1.0中引入了MIME协议，所以在HTML中可以展示除了文本之外的其他格式的东西，如下：



![image-20181014001053415](/Users/chenyansong/Documents/note/images/http/image-20181014001053415.png)



上面 浏览器请求了一张图片，然后服务器返回了这张图片，从response中返回的字段中，我们可以看到Content-Type=imge/jpeg这个返回数据的格式是图片，具体的图片是jpeg的格式，**于是浏览器调用图片的插件来显示图片**



动态网页：服务器端存储的文档非HTML格式，二十编程语言开发的脚本，脚本接收参数之后，在服务器端执行一次，运行完成之后会生成一个HTML格式的文档，服务器把生成的文档发给客户端。





# 常见的http客户端和服务器端

* Client
  * IE
  * Firefox
  * Chrome
  * Opera
  * Safari
* Server（web服务器，仅处理静态内容）
  * Apache—>httpd
  * IIS(web服务器，应用程序服务器)
  * nginx
  * lighttpd
  * thttpd (嵌入式使用)
* 应用程序服务器（能够解析静态内容，并且处理某种特定格式的动态内容）
  * IIS
  * Tomcat（Java应用程序服务器， open source）
  * Websphere(IBM, jsp解析, 商业)
  * Weblogic(Bea, Oracle, jsp, 商业)
  * JBoss(RedHat, 开源 or 商业)