---
title: document的核心元数据
categories: elasticsearch   
toc: true  
tag: [elasticsearch]
---




1、_index元数据
2、_type元数据
3、_id元数据

```
{
  "_index": "test_index",
  "_type": "test_type",
  "_id": "1",
  "_version": 1,
  "found": true,
  "_source": {
    "test_content": "test test"
  }
}
```

--

1、_index元数据

（1）代表一个document存放在哪个index中
（3）index中包含了很多类似的document：类似是什么意思，其实指的就是说，这些document的fields很大一部分是相同的，你说你放了3个document，每个document的fields都完全不一样，这就不是类似了，就不太适合放到一个index里面去了。
（4）索引名称必须是小写的，不能用下划线开头，不能包含逗号：product，website，blog

2、_type元数据

（1）代表document属于index中的哪个类别（type）
（2）一个索引通常会划分为多个type，**逻辑上对index中有些许不同的几类数据进行分类**：因为一批相同的数据，可能有很多相同的fields，但是还是可能会有一些**轻微的不同**，可能会有少数fields是不一样的，举个例子，就比如说，商品，可能划分为电子商品，生鲜商品，日化商品，等等。
（3）type名称可以是大写或者小写，但是同时不能用下划线开头，不能包含逗号

3、_id元数据

（1）代表document的唯一标识，与index和type一起，可以唯一标识和定位一个document
（2）我们可以手动指定document的id（put /index/type/id），也可以不指定，由es自动为我们创建一个id




