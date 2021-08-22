[toc]



模型层：负责与数据库进行通信

# 准备

* mysql lib

  mysqlclient 1.3.13+

  ```shell
  sudo apt list --installed|grep -E 'libmysqlclient-dev|python3-dev'
  
  #安装
  sudo apt-get install python3-dev default-libmysqlclient-dev
  
  sudo pip3 install mysqlclient #他会依赖上面的安装
  
  #查看是否安装
  sudo pip3 freeze|grep -i 'mysql'
  ```



# 配置

* 创建数据库

  ```shell
  create database db_name default charset utf8
  ```

  

* 修改setting配置

  ```python
  #修改DATABASES项
  
  DATABASES = {
      'default':{
          'ENGINE':'django.db.backends.mysql',
          'NAME':'db_name',
          'USER':'root',
          'PASSWORD':'123',
          'HOST':'LOCALHOS',
          'PORT':'3306'
      }
  }
  
  
  #可选的数据库引擎
  django.db.backends.mysql
  django.db.backends.sqlite3
  django.db.backends.oracle
  django.db.backends.postgresql
  ```



# Model

* django.db.models.Model 派生出的子类，即只需要继承这个类即可成为模型
* 一个model类代表的是数据库中的一张表
* model中的每个属性代表数据库表中的一个字段



# ORM框架

```python
#file:bookstore/models.py
#需要继承Model
class Book(models.Model):
    title = models.CharField("书名", max_length=50, default='')
    
#默认情况下，table_name = app_modelName : bookstore_book

#任何表结构的修改，务必在对应的模型类上修改，优劣Django的Model，就不要直接操作数据库了
```



# 数据库迁移

```python
#迁移是Django同步模型所做的更改（添加字段，删除模型等）到数据库中

#1.生成迁移文件(将APP下的models.py文件生成一个中间文件，并保存在migrations文件夹中)
python3 manage.py makemigrations
#2.执行迁移脚本程序（将每个应用下的migrations目录下的中间文件同步回数据库中）
python3 manage.py migrate
```



# 基础字段及选项

```python
#BooleanField ---> tinyint(1)
#在Django中使用的是TRUE，False
#在数据库中存的是1,0

#CharField ---> varchar
#注意此时必须指定max_length 选项

#DateField ---> date
#参数：
auto_now = True/False #TRUE每次保存对象的时候，自动设置该字段的值为当前时间
auto_now_add = True/False #当对象第一次被创建的时候自动设置当前时间
default #设置当前时间（字符串格式时间如：‘2019-6-1’
# 三选一

#DateTimeField ----> datetime
#参数同上


#FloatField ---> double

#DecimalField --> decimal(x,y)
#max_digits 总位数
#decimal_places #小数点后几位

#EmailField --->varchar

#IntegerField --> int

#ImageField ---> varchar(100)
#存的是图片的存储路径

#TextField ---> longtext
#不定长的大文本


#example
class Author(models.Model):
    name = models.CharField("name", max_length=11)
    age = models.IntergerField("age")
    email = models.EmailField("email")
```









