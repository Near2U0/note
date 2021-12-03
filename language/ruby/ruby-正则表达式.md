ruby-正则表达式

http://rubyer.me/blog/357/



```python
def ipnumber(ip):
    ip=ip.rstrip().split('.')
    ipn=0
    while ip:
        ipn=(ipn<<8)+int(ip.pop(0))
    return ipn

def ipstring(ip):
    ips=''
    for i in range(4):
        ip,n=divmod(ip,256)
        ips = str(n)+'.'+ips
    return ips[:-1] ## take out extra point
```





# intIP和stringIP互相转换

```ruby
require 'ipaddr'

IPAddr.new("192.168.0.1").to_i
 => 3232235521 

IPAddr.new(3232235521, Socket::AF_INET).to_s
 => "192.168.0.1" 

https://stackoverflow.com/questions/33883365/can-i-store-ip-addresses-as-integers
```

