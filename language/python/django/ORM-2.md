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

django_migrations表记录了migrate的全过程，项目各应用中的migrate文件与之对应，在多人协作时可能会遇到文件不一致的情况：解决如下：

1. 删除所以的migrations里戴尔所以的000xx.py(__init__.py 除外)

2. 删除数据库(本地的测试数据库)

   sql>drop database mywebdb

3. 重新创建数据库

   sql>create dabase mywebdb default charset utf-8

4. 重新migrations 

   python3 manage.py makemigrations

5. 重新migrate

   python3 manage.py migrate





# 基础字段

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

# 选项

指定创建的列的额外的信息

```python
#primary_key True 表示该列是主键，如果指定了一个字段为主键，那么将不会创建ID字段了

#blank 设置为True时，字段可以为空，设置为False,字段必须填写，这个跟MySQL的NULL不是一个意思，此处是给Django的管理后台使用

#null 设置为True，表示该列允许为空，默认是Fasle,如果此项设置为FALSE，建议加入default选项来设置默认值

#default #设置所在列的默认值，如果null=False,建议添加此项，对于新增的字段，如果不写这个属性，他会提示你是现在给默认值，还是代码中给默认值，所以最好是加上

#db_index 如果设置True，表示为此列添加索引

#unique 表示该列的字段必须唯一

#db_column 指定列的名称，如果不指定的话，使用属性名作为列名

#verbose_name 设置此字段在admin界面上的显示名称

name = models.CharField(max_length=30,
                       unique=True, null=False, db_index=True)

```



> 好习惯，修改过的字段选项，【添加or更改】均要执行makemigrations 和migrate





# 模型类-Meta类

meta类是修改的表的属性信息

```python
class Book(models.Model):
    title = models.CharField("title", max_length=50,default='')
    class Meta:
        db_table = "book" #修改表名
```





# 创建数据



* 管理器对象

  每个继承自models.Model的模型类，都会有一个objects对象被同样继承下来，这个对象叫管理器对象，数据库的增删改查可以通过模型的管理器实现

  ```python
  class MyModel(model.Model):
      #...
  MyModel.objeccts.create(...) #objects是管理器对象
  ```

  

* 创建数据

  ```python
  #方式1
  MyModel.objects.create(属性1=值1， 属性2=值2, ...)
  #成功返回创建好的实体对象
  #失败，抛出异常
  
  #方式2
  #创建MyModel实例对象，调用save()
  obj = MyModel(属性=值，属性=值)
  obj.属性=值
  obj.save()
  ```

* django shell

  ```shell
  # 项目代码发生变化时，重新进入Django shell
  python3 mange.py shell
  
  
  ```

  











