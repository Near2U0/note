转：https://blog.csdn.net/qq_39147299/article/details/108540488



如果返回给前端的数据如果很重要，我们可以使用itsdangerous等进行加密传输。但并不是很敏感，我们可以使用简单的编码不让它直接显示明文，比如说用pickle序列化成字节，再使用base64编码成一个可视化的字符串

1.编码和解码
pickle编码解码只需要调用dumps()和loads()方法即可，base64编码解码只需要调用b64encode()和b64decode()即可
需要注意的是，编码得到的数据是字节类型的
————————————————



```python
import pickle,base64

data = {"name":"pan"}

# pickle编码
data_bytes = pickle.dumps(data)
print(data_bytes)  # b'\x80\x03}q\x00X\x04\x00\x00\x00nameq\x01X\x03\x00\x00\x00panq\x02s.'

# base64编码
data_str = base64.b64encode(data_bytes)
print(data_str)  # b'gAN9cQBYBAAAAG5hbWVxAVgDAAAAcGFucQJzLg=='

# base64解码
data_bytes2 = base64.b64decode(data_str)
print(data_bytes2) # # b'\x80\x03}q\x00X\x04\x00\x00\x00nameq\x01X\x03\x00\x00\x00panq\x02s.'

# pickle解码
data2 = pickle.loads(data_bytes2)
print(data2) # data = {"name":"pan"}

```







