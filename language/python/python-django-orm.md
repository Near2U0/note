



# [django ORM 多条件查询技巧1](https://my.oschina.net/waston/blog/597152)

原创

[原谅我的突然](https://my.oschina.net/waston)

[工作日志](https://my.oschina.net/waston?tab=newest&catalogId=3411924)

2016/01/04 21:58

阅读数 5.6K

```
# 获取动态过滤调价
def getKwargs(data={}):
     kwargs = {}
     kwargs['state'] = True 
     for (k , v)  in data.items() :
        if v is not None and v != u'' :
                 kwargs[k] = v          
        return kwargs
#然后使用的时候：        
searchCondition = {'name__icontains' : name ,....}
kwargs = utils.getKwargs(searchCondition)
model_set = Model.objects.filter(**kwargs）
```

