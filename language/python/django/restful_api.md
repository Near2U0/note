



# 序列化的问题

这个链接处理的是传统json参数返回和restful-json的返回的方式的比较

https://cloud.tencent.com/developer/article/1506248





# 查询参数获取的问题

https://www.jianshu.com/p/4b746e566d78

```python

def get_parameter_dic(request, *args, **kwargs):
    if not isinstance(request, Request):
        return {}

    query_params = request.query_params
    if isinstance(query_params, QueryDict):
        query_params = query_params.dict()
    result_data = request.data
    if isinstance(result_data, QueryDict):
        result_data = result_data.dict()

    if query_params != {}:
        return query_params
    else:
        return result_data

    
    
#request调用
class IndexView(APIView):
    """查询详情页"""
    def get(self, request, *args, **kwargs):
        status = STATUS.OK
        msg = "error"
        try:
            params = get_parameter_dic(request)
            print(params)
            index_all = Index.objects.all()
            index_data = serializers.IndexSearchSerializer(index_all, many=True).data
            return json_response(status=status, msg="Success", data=index_data)
        except Exception as e:
            status = STATUS.ERR
            log_utils.log_error("index data search err %s" % e)
        return json_response(status=status, msg=msg, data=None)
```



```shell
这个 sqlmigrate 命令并没有真正在你的数据库中的执行迁移 - 它只是把命令输出到屏幕上，让你看看 Django 认为需要执行哪些 SQL 语句。这在你想看看 Django 到底准备做什么，或者当你是数据库管理员，需要写脚本来批量处理数据库时会很有用。
如果你感兴趣，你也可以试试运行 python manage.py check ;这个命令帮助你检查项目中的问题，并且在检查过程中不会对数据库进行任何操作。


让我们看看迁移命令会执行哪些 SQL 语句。sqlmigrate 命令接收一个迁移的名称，然后返回对应的 SQL：

$ python manage.py sqlmigrate polls 0001
```





# 序列化

```
ManyToManyField

https://blog.csdn.net/hpu_yly_bj/article/details/78941104
```



manytomany时，需要展示的数据是多行

