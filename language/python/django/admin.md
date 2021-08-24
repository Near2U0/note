# admin配置

```python
python manage.py createsuperuser


```



# 注册自定义的模型类

```python
#1.APP中的admin.py
from .models import Book
#2.调用admin.site.register方法进行注册
admin.site.register(自定义的模型类)
```





