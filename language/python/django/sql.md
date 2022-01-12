

# values与 values_list

reference：https://www.cnblogs.com/rgxx/p/10382664.html

* values

```python
>>> Blog.objects.values()
[{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}],
>>> Blog.objects.values('id', 'name')
[{'id': 1, 'name': 'Beatles Blog'}]

#同时需要序列化为json格式：使用list() 转换
     obj_list = models.DeviceCategory.objects.filter(**query_params).values('id', 'parent_id', 'device_name')
            return json_response(status=status, msg="Success", data=list(obj_list))
    
```

* values_list

```python
>>> Entry.objects.values_list('id').order_by('id')
[(1,), (2,), (3,), ...]

>>> Entry.objects.values_list('id', flat=True).order_by('id')
[1, 2, 3, ...]


```





