# mongodb报错TypeError:ObjectId("**********") is not JSON serializable



在使用flask 做一个关于Mongo的查询的时候，出现了标题中的错误
原因是查询到的结果集中的_id字段是一个ObjectId对象，无法转化为json

两个解决办法:

在查询语句中直接去掉_id字段

```
def query_all(limit, page):
    count = mongo.db.student.find({}).count()
通过{"_id": 0}直接去掉_id字段
    online_students = mongo.db.student.find({}, {"_id": 0}).limit(int(limit)).skip((page - 1) * int(limit))
    return count, online_students
```



你真的需要返回_id字段到前端页面
经过百度找到了一个解决办法
废话不多少 直接上解决办法

```
import json
from bson import json_util
```



导入上述模块
然后将查询出的结果集做一个转化，然后在将结果jsonify()的时候就不会报错了!

data就是你用查询语句查出来的结果集
```shell
json.loads(json_util.dumps(data))
```




参见：https://blog.csdn.net/qq_30500113/article/details/102842460

