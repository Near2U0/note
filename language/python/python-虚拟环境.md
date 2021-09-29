[toc]



# install env

## install virtualenv

```shell
pip install virtualenv



virtualenv venv　　#venv为虚拟环境目录名，目录名自定义,virtualenv venv 将会在当前的目录中创建一个文件夹，包含了Python可执行文件，以及 pip 库的一份拷贝，这样就能安装其他包了。虚拟环境的名字（此例中是 venv ）可以是任意的；若省略名字将会把文件均放在当前目录。

#在任何你运行命令的目录中，这会创建Python的拷贝，并将之放在叫做 venv 的文件中。
#你可以选择使用一个Python解释器：

$ virtualenv -p /usr/bin/python2.7 venv　　　　# -p参数指定Python解释器程序路径

```



## install virtualenvwrapper

　　安装virtualenvwrapper(确保virtualenv已安装)

```
pip install virtualenvwrapper
pip install virtualenvwrapper-win　　#Windows使用该命令
```



## virtualenvwrapper基本使用

1. 创建虚拟环境　**mkvirtualenv**

```
mkvirtualenv venv
```

　　这样会在WORKON_HOME变量指定的目录下新建名为venv的虚拟环境。

　　若想指定python版本，可通过"--python"指定python解释器

```
mkvirtualenv --python=/usr/local/python3.5.3/bin/python venv
```

2. 基本命令 

   查看当前的虚拟环境目录

```
[root@localhost ~]# workon
py2
py3
```

　　切换到虚拟环境

```
[root@localhost ~]# workon py3
(py3) [root@localhost ~]# 
```

　　退出虚拟环境

```
(py3) [root@localhost ~]# deactivate
[root@localhost ~]# 
```

　　删除虚拟环境

```
rmvirtualenv venv
```

 



# create project in env

```shell

#进入虚拟环境
workon env-demo

#在虚拟环境中安装Django
pip install django


# 初始化工程
cd /path/to/project/
django-admin startproject project-demo

```



