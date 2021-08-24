



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



