# wsgi协议实现

> brief: 实现的原理，接收来自socket的数据流，然后将socket的数据转换为基于http的请求的数据，提交给后端的application，同时传一个http_response给后端，接收后端的response，也是将后端的响应数据通过socket封装之后发送回client





![](../../..\images\linux\kernel\network\wsgi.png)



以下是一个wsgi协议的实现

```python
# *-* coding: utf-8 *-*
 
import socket
import StringIO 
 
class WSGIServer(object):
    address_family = socket.AF_INET 
    socket_type = socket.SOCK_STREAM 
    request_queue_size = 5
 
    def __init__(self, address):
        self.socket = socket.socket(self.address_family, self.socket_type)
        self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.socket.bind(address)
        self.socket.listen(self.request_queue_size)
        self.host, self.port = self.socket.getsockname()
        self.server_name = socket.getfqdn(self.host)
 
    def set_app(self, application):
        self.application = application 
 
    def server_forever(self):
        print "start listting"
        while 1:
            self.connection, client_address = self.socket.accept()
            print "recive a request!"
            self.request_handler()
 
    def request_handler(self):
        self.request_data = self.connection.recv(1024)
        self.parse_request()
        env = self.get_env()
        result = self.application(env, self.start_response)
        self.finish_response(result)
 
    def parse_request(self):
        request_data = self.request_data.splitlines()
        request_line = request_data[0].strip('\r\n')
        self.request_method, self.request_path, self.request_version = request_line.split()
 
    def get_env(self):
        env = {}
        # WSGI必要参数
        env['wsgi.version']      = (1, 0) 
        env['wsgi.url_scheme']   = 'http' 
        env['wsgi.input']        = StringIO.StringIO(self.request_data) #返回一个
        env['wsgi.errors']       = sys.stderr 
        env['wsgi.multithread']  = False 
        env['wsgi.multiprocess'] = False 
        env['wsgi.run_once']     = False 
        ### CGI 必需变量 
        env['REQUEST_METHOD']    = self.request_method    # GET 
        env['PATH_INFO']         = self.request_path      # 请求路径 
        env['SERVER_NAME']       = self.server_name       # localhost 
        env['SERVER_PORT']       = str(self.port)  # 8888 
        return env
 
    def start_response(self, status, response_headers):
        '''
            this function is used to set response_headers and status 
        ''' 
        self.headers = [status, response_headers]
 
    def finish_response(self, result):
        try:
            status, headers = self.headers
            response = 'HTTP/1.1 {status}\r\n'.format(status=status)
            for h in headers:
                response += '%s:%s\r\n'%(h)
            response += '\r\n'
            for data in result:
                print data 
                response += data 
            self.connection.sendall(response)
        finally:
            self.connection.close()
 
ADDRESS = ('localhost', 8888)
if __name__ == '__main__':
    import sys 
    if len(sys.argv) < 2:
        sys.exit('must give module:callable') 
    app_path = sys.argv[1]
    path, app = app_path.split(':')
    module = __import__(path)
    app = getattr(module, app)
    server = WSGIServer(ADDRESS)
    #设置后端的application
    server.set_app(app)
    #接收请求
    server.server_forever()
    
```



**WSGI / uwsgi / uWSGI 这三个概念的区分。**

* WSGI，是一种描述web服务器（如nginx，uWSGI等服务器）如何与web应用程序（如用Django、Flask框架写的程序）**通信协议**。
* uwsgi协议是一个uWSGI服务器自有的协议，它用于定义传输信息的类型（type of information），每一个uwsgi packet前4byte为传输信息类型描述，用于与nginx等代理服务器通信，它与WSGI相比是两样东西。
* uWSGI是实现了uwsgi和WSGI两种协议的**Web服务器**。



![](../../..\images\linux\kernel\network\wsgi_2.png)



为什么有了uWSGI为什么还需要nginx？因为nginx具备优秀的静态内容处理能力，然后将动态内容转发给uWSGI服务器，这样可以达到很好的客户端响应。





参考：https://blog.csdn.net/dream_is_possible/article/details/79670774

https://blog.csdn.net/weixin_45455015/article/details/100113330



