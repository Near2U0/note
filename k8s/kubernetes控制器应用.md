[TOC]

# pod-yaml定义



Pod资源

```shell
spec.containers <[]object>
- name <string>
	image <string>
	
	imagePullPolicy <string> #这里可以指定镜像仓库的地址
		Always(总是从仓库中下载:如果我们想要image，刚好在本地存在，但是这个镜像是伪装的，并不是我们自己的), Never(如果本地有就用，没有也不会从仓库下载), IfNotPresent(如果本地存在直接使用，否则就从仓库下载)
		#如果镜像的标签是latest，那么使用的是Always
		#否则使用的是IfNotPresent
		
	ports <[]object> #这里只是显示的说明暴露了哪些端口
  	containerPort <integer>
  	name <string>
  	protocol <string>
  	
  command <[]string> #如果没有提供，那么将会运行image中的entrypoint
  
  args <[]string> #代替镜像的CMD传递给entrypoint的参数，通过$(var_name) 作为变量的引用
		
```

![image-20190720215028438](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190720215028438.png?raw=true)

![image-20190720215250060](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190720215250060.png?raw=true)

一个资源可以拥有多个标签，而一个标签可以被添加到多个对象上

```shell
key=value
	key:只能是数字，字母，_, -,., 并且只能字母或数字开头及结尾
	value:可以为空，只能是数字，字母，_, -,., 并且只能字母或数字开头及结尾
	
```

```shell
#查看pod的标签
kubectl get pods --show-labels
```

![image-20190720222258006](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190720222258006.png?raw=true)

```shell
#过滤标签:查看有app标签
kubectl get pods -l app

#显示具体的标签是什么
kubectl get pods -l app --show-labels
#显示同时又app,run标签的pod
kubectl get pods -L app,run

#显示每个pod的对应标签的标签的值
kubectl get pods -L app,run
```

![image-20190720222932994](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190720222932994.png?raw=true)



修改资源的标签

![image-20190720223051545](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190720223051545.png?raw=true)

```shell
kubectl label pods pod-demo release=canary
```

![image-20190720223146251](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190720223146251.png?raw=true)

修改已经存在的标签

```shell
kubectl label pods pod-demo release=stable --overwrite
```

![image-20190720223359085](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190720223359085.png?raw=true)



标签选择器：

1. 等值关系的：= or != 

2. 集合关系的: 

   key in (val1, val2….) 

   key notin (val1, val2...)

   key ： 存在这个key就行

   !key : 不存在此键

3. 许多资源支持内嵌字段定义其使用的标签选择器

   matchLabels：直接给定键值对

   matchExpressions:基于给定的表达式来定义使用的标签选择器 {key:"keyName", operator:"=", values:[val1, val2...]}

   ​	operator:

   ​		In， NotIn：values必须为非空列表

   ​		Exists, NotExists：values的值必须为空列表

```shell
#等值关系
kubectl get pods -l release=stable --show-labels
kubectl get pods -l release=stable,app=myapp --show-labels

kubectl get pods -l release !=stable

#集合关系
kubectl get pods -l "release in (canary,beta,alpha)"

kubectl get pods -l "release notin (canary,beta,alpha)"

```



节点也是可以打标签的

```shell
#查看节点的标签
kubectl get nodes --show-labels

#给node打标签
kubectl label nodes node01.test.com disktype=ssd
```

![image-20190721081945136](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721081945136.png?raw=true)

```shell
nodeSelector <map[String] string>	节点标签选择器

nodeName <String> 运行在指定节点上
```

![image-20190721082319642](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721082319642.png?raw=true)

因为node01上有这个标签，所以我们重新创建pod的时候，这个pod是运行在node01上的

![image-20190721082632829](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721082632829.png?raw=true)



```shell
annotations:注解
#与label不同的地方在于，他不能英语挑选资源对象，仅用于为对象提供“元数据”

#查找pod的annotations
kubectl describe pods pod-demo
```

![image-20190721083118944](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721083118944.png?raw=true)

为资源添加annotation

![image-20190721083339583](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721083339583.png?raw=true)

之后再次通过yaml文件创建Pod，然后describe查看

![image-20190721083441202](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721083441202.png?raw=true)



Pod的生命周期

状态：

​	Pending 调度尚未完成

​	Running 运行状态

​	Failed	失败

​	Succeeded

​	Unkown 为止

![image-20190721085119829](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721085119829.png?raw=true)

容器的重启策略

```shell
restartPolicy
# Always  ： default
#OnFailure： when failure ,will restart
#Never
```



探针类型三种：

```shell
ExecAction
TCPSockcetAction
HTTPGetAction

#查看container的探针
#kubectl explain pods.spec.containers

```

![image-20190721091732018](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721091732018.png?raw=true)

livenessProbe：存活性探测

```shell
kubectl explain pods.spec.containers.livenessProbe
#他下面可以定义这三种探针
ExecAction
TCPSockcetAction
HTTPGetAction

#探测几次
failureThreshold <integer>
#默认是探测3次，3次之后，就返回失败

#在liveness 探测的延迟探测时间，需要等待的时间(等待初始化完成)
initialDelaySeconds

#每次间隔的时长
periodSeconds <integer>
#默认是10秒探测一次

#每次探测如果没有响应，需要等待的时长，默认是1s
timeoutSeconds

```

下面就exec探针进行说明

```shell
kubectl explain pods.spec.containers.livenessProbe.exec

#返回0表示healthy, 返回非0白鸥是unhealthy
```

![image-20190721102840091](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721102840091.png?raw=true)

tcpSocket

![image-20190721103415928](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721103415928.png?raw=true)



httpGet

![image-20190721103509385](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721103509385.png?raw=true)

![image-20190721103825924](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721103825924.png?raw=true)

我们连接进入上面的Pod的容器

```shell
kubectl -it liveness-httpget-pod -- /bin/sh
#手动删除文件
```

![image-20190721104120338](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721104120338.png?raw=true)



readinessProbe

![image-20190721122647088](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721122647088.png?raw=true)

我们删除readinessProbe探测的文件

![image-20190721122832459](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721122832459.png?raw=true)

我们再次查看创建pod的状态

![image-20190721122921648](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721122921648.png?raw=true)

我们进入pod内部，看NGINX服务是存在的

![image-20190721123127168](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721123127168.png?raw=true)

我们重新创建删除的文件

![image-20190721123157622](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721123157622.png?raw=true)

我们可以看到pod又就绪了

![image-20190721123216746](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721123216746.png?raw=true)



启动后钩子和终止前钩子

```shell
kubectl explain pod.spec.containers.lifecycle

```

![image-20190721123530982](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721123530982.png?raw=true)

对于启动后，或者是终止前，我们可以看到也是有三种探针

![image-20190721123621336](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721123621336.png?raw=true)

我们创建一个postStart的Pod

![image-20190721125857115](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721125857115.png?raw=true)



进入容器内部，查看是否创建了目录

![image-20190721125529545](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721125529545.png?raw=true)



# Pod控制器

* ReplicaSet： (ReplicationController的新一代，推荐使用)，不是我们直接使用的控制器

  满足用户数量的副本

  标签选择器：选择由自己管理和控制的副本

  根据Pod资源模板来完成Pod的新建

  扩缩容操作

* Deployment(无状态)：建构在ReplicaSet之上，通过ReplicaSet来控制Pod

  ReplicaSet的功能

  滚动更新

  回滚机制

  申明式配置：随时改变资源的运行状态

* DaemonSet(无状态)：集群中的每个节点只会运行一个特定的Pod副本
* Job：只是为了完成某项任务，完成之后pod推出
* CronJob:周期性的Job
* StatefulSet：有状态controller



## RelicaSet

简称rs

```shell
#查看文档
kubectl explain rs
```

![image-20190721210233497](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721210233497.png?raw=true)

我们来定义一个rs,在上面我们使用yaml文件去定义了一个Pod，但是这种Pod是不受controller管理的，所以如果我们要用controller去管理Pod，可以在controller中来定义Pod，定义的格式和Pod定义很相似

![image-20190721211821480](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721211821480.png?raw=true)

然后我们根据定义去创建rs

```shell
kubectl create -f rs-demo.yaml

#查看控制器创建的Pod
#在pod中定义的名称是没有用的，他是以控制器的名称加一个字符串组成
```

![image-20190721212049630](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721212049630.png?raw=true)

我们删除一个Pod副本，controller会自动重建

我们的rs控制器定义的副本数量为2，但是如果我们一个不相关的pod的标签，将rs中通过标签选择器选中的的Pod变为3，我们看看结果会如何



![image-20190721212552053](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721212552053.png?raw=true)

![image-20190721212745013](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721212745013.png?raw=true)

我们发现rs会删除一个Pod，所以我们在定义标签的时候，需要复杂定义，避免冲突

假如我们想要修改Pod的副本

```shell
kubectl edit rs myapp
#我们将副本改成5
```

![image-20190721213818894](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721213818894.png?raw=true)

我们再查看副本数量

![image-20190721213859278](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721213859278.png?raw=true)

然后我们修改Pod的image的版本，从v1改为v2

![image-20190721214231139](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721214231139.png?raw=true)

我们看到现有的Pod的版本还是v1，因为只有重建的Pod，才会是v2版本的

![image-20190721214408855](https://github.com/chenyansong1/note/blob/master/images/docker/image-20190721214408855.png?raw=true)

## Deployment

我们使用Deployment帮我们自动实现上述过程

![1563764257495](E:\git-workspace\note\images\docker\1563764257495.png)

![1563774855399](E:\git-workspace\note\images\docker\1563774855399.png)

![1563775166188](E:\git-workspace\note\images\docker\1563775166188.png)

Deployment还可以控制更新节奏和更新逻辑（根据流量请求），允许多出来m个：

1. 最少5个，可以允许多出来k个
2. 最多5个，可以少出来k个
3. 最大允许多m1个，最少允许少m2个

```shell
#
kubectl explain deployment

kubectl explain deployment.spec
#更新策略
kubectl explain deployment.spec.strategy
#滚动更新
kubectl explain deployment.spec.strategy.rollingUpdate
   maxSurge       <string>     #最多有多少个pod；可以是数量or百分比
   maxUnavailable       <string> #最多多少个不可用;可以是数量or百分比

#保存多少个历史版本
kubectl explain deployment.spec.revisionHistoryLimit


#创建定义Pod的模板
kubectl explain deployment.spec.template

```

下面是我们定义的一个Deployment

![1563776972242](E:\git-workspace\note\images\docker\1563776972242.png)

```shell
#apply表示一种申明式更新，申明式创建的命令
kubectl apply -f deploy-demo.yaml
```

![1563777126093](E:\git-workspace\note\images\docker\1563777126093.png)

因为deployment是构建在replicaSet之上的，所以我们也是可以看到rs的存在的

![1563777183623](E:\git-workspace\note\images\docker\1563777183623.png)

> 69b47..是template的hash值

我们查看生成的Pod

![1563777254217](E:\git-workspace\note\images\docker\1563777254217.png)

现在我们想要改变副本的数量，我们直接编辑配置文件即可，在配置文件中将副本数改为3，然后`kubectl apply -f deploy-demo.yaml`

![1563777412745](E:\git-workspace\note\images\docker\1563777412745.png)

默认是滚动更新，都是25%的更新

![1563778845940](E:\git-workspace\note\images\docker\1563778845940.png)

打补丁

![1563792404953](E:\git-workspace\note\images\docker\1563792404953.png)

查看Pod

![1563792438667](E:\git-workspace\note\images\docker\1563792438667.png)

```shell
#最多不可用0，最大可用数量为6
#kubectl rollout pause --help

kubectl rollout pause --help
#打补丁
kubectl patch deployment myapp-deploy -p '{"spec":{"strategy":{"rollingUpdate":{"maxSurge":1, "maxUnavailable":0}}}}'
```

![1563792837818](E:\git-workspace\note\images\docker\1563792837818.png)

回滚

```shell
kubectl rollout undo --help

#查看版本
kubectl rollout history deployment myapp-depoy
```

![1563796819431](E:\git-workspace\note\images\docker\1563796819431.png)

```shell
#回滚到第一版
kubectl rollout undo deployment myapp-deploy -to-revision=1
```

![1563796895662](E:\git-workspace\note\images\docker\1563796895662.png)



## DaemonSet

```shell
kubectl explain ds
```

![1563797982115](E:\git-workspace\note\images\docker\1563797982115.png)

```shell
kubectl apply -f ds-demo.yaml
```

在一个yaml中定义多个资源`使用---隔开每个资源`

![1563844695501](E:\git-workspace\note\images\docker\1563844695501.png)

