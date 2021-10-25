# docker启动报错：standard_init_linux.go:211: exec user process caused "no such file or directory"



     如题所示，根据自己构建的镜像启动docker容器，直接退出，查看容器日志报错信息，没有任何别的信息。网上搜索这个问题，发现很多人都遇到过，解决办法也各不相同，最后发现一篇文章。受到启发，我的项目是java项目，通过ENTRYPOINT命令启动脚本docker-entrypoint.sh来构建一个在后台运行的服务。而我的docker-entrypoint.sh是在windows下编辑的，自然fileformat是dos，这里需要修改为unix，修改办法也很简答，无需再在linux下操作，我们一般机器上安装了git工具，自带了git bash命令行工具，进入git bash，找到该文件docker-entrypoint.sh，然后使用vi编辑，修改fileformat=unix，如下所示。
![img](..\images\docker\docker-ff.png)