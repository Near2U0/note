[TOC]



docker-compose是一个单机的编排工具





```shell



#如果容器映射不生效，那么可以尝试如下的重启方式
docker-compose up -d		#-d 后台运行


  pull               Pull service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  top                Display the running processes
  unpause            Unpause services
  up                 Create and start containers (重新创建容器，有时候修改不生效的时候用)
  version            Show the Docker-Compose version information


```

构建镜像

```shell
docker build . -t secevent:v1.0
```



docker-compose启动

```shell
docker-compose up -d 
```

