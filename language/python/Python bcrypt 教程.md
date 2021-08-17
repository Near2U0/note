

https://geek-docs.com/python/python-tutorial/python-bcrypt.html



```python
#!/usr/bin/env python3

import bcrypt

passwd = b's$cret12'

salt = bcrypt.gensalt()
hashed = bcrypt.hashpw(passwd, salt)

print(salt)
print(hashed)
```



密码检查

```python
#!/usr/bin/env python3

import bcrypt

passwd = b's$cret12'

salt = bcrypt.gensalt()
hashed = bcrypt.hashpw(passwd, salt)

if bcrypt.checkpw(passwd, hashed):
    print("match")
else:
    print("does not match")
```

