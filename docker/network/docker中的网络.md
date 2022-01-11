

转自：https://www.cnblogs.com/JulianHuang/p/10919346.html



## 网桥模式

   最后我们来探究容器集合的[网络连接](https://docs.docker.com/compose/networking/)， 这也是容器比较复杂的部分。

 docker引擎刚建立的时候，会新建一个docker0网桥（driver= bridge）， 新加入的容器默认都会接入这个网桥。

 ![img](https://img2018.cnblogs.com/blog/587720/201905/587720-20190509122254701-897743225.gif) 当执行docker-compose up时，会创建新的网桥设备，集合内所有容器都通过该网桥交流：

① 创建名为 {project}_default 的网桥

② 以服务名app加入 {project}_default 网络； 以服务名nginx加入 {project}_default 网络

   *每一个容器现在可使用 “app” / “nginx” 服务名作为主机名相互访问*

> **为啥可以通过 服务名访问 容器？  **
>
> 是因为利用了 Docker引擎内置的DNS， 查询服务名----》 查询DNS（每个服务名： 对应容器IP） 

   所以在nginx.conf 文件中我们给 【upstream app_servers】配置 app:80 能正确转发请求：



