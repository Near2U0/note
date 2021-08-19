sqlalchemy批量删除数据、全量删除

```shell
问题：sqlalchemy如何批量删除多条数据
解决：使用参数synchronize_session=False，或for循环
方法：
        users = self.db.query(User).filter(User.id.in_(1,2,3)).all()
        [self.db.delete(u) for u in users]
        self.db.commit()
或
        users = self.db.query(User).filter(User.id.in_(1,2,3)).delete(synchronize_session=False)
        self.db.commit()

 

全量删除搜索到的：删除所有家是上海的用户的信息

self.db.query(User).filter(User.home=='shanghai').delete()


```

