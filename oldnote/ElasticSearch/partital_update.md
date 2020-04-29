---
title: partital_update
categories: elasticsearch   
toc: true  
tag: [elasticsearch]
---



# 1.一般的更新语法

PUT /index/type/id，创建文档&替换文档，就是一样的语法

一般对应到应用程序中，每次的执行流程基本是这样的：

（1）应用程序先发起一个get请求，获取到document，展示到前台界面，供用户查看和修改
（2）用户在前台界面修改数据，发送到后台
（3）后台代码，会将用户修改的数据在内存中进行执行，然后封装好修改后的全量数据
（4）然后发送PUT请求，到es中，进行全量替换
（5）es将老的document标记为deleted，然后重新创建一个新的document

# 2.什么是partial update？

```
partial update

post /index/type/id/_update 
{
   "doc": {
      "要修改的少数几个field即可，不需要全量的数据"
   }
}

```

> 看起来，好像就比较方便了，每次就传递少数几个发生修改的field即可，不需要将全量的document数据发送过去


# 3.图解partial update实现原理以及其优点

partial update，看起来很方便的操作，实际内部的原理是什么样子的，然后它的优点是什么

![](/Users/chenyansong/Documents/note/images/es/图解partial_update实现原理以及其优点.png)

```
#一般的put
PUT /test_index/test_type/10
{
  "test_field1": "test1",
  "test_field2": "test2"
}

#局部更新
POST /test_index/test_type/10/_update
{
  "doc": {
    "test_field2": "updated test2"
  }
}

```

如果更新的文档不存在，那么会出现异常

# retry_on_conflict

```
post /index/type/id/_update?retry_on_conflict=5&version=6
```

![](/Users/chenyansong/Documents/note/images/es/partial update内置乐观锁并发控制.png)


