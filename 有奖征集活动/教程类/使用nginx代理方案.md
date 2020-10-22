
# Fate  使用 nginx 转发方案

#### Fate 使用的 grpc 框架，底层是基于 TCP 长连接进行数据交互达成， 因此使用 nginx 转发的时候，一般的 http 请求转发模式是不可取的，Nginx 需要支持 tcp 协议。

#### Nginx1.9.9 及以上便可支持 tcp 请求转发，值得注意的是，安装的时候需要 携带参数–with-stream 等安装，否则亦无法使用。

#### 具体过程如下：

##### 安装 nginx:
```
ps -ef|grep nginx
service nginx stop
ps -ef|grep nginx|awk '{print $2}'|xargs kill -9
service nginx stop
yum remove nginx
yum install -y gcc-c++ pcre* openssl*
cd /usr/local/
wget http://nginx.org/download/nginx-1.9.9.tar.gz
tar -zxvf nginx-1.9.9.tar.gz
cd nginx-1.9.9
./configure --prefix=/usr/local/nginx/ --with-http_stub_status_module
--with-http_ssl_module --with-stream --with-stream_ssl_module
make
make install
cd ../nginx
vim ./conf/nginx.conf
./sbin/nginx
```


#### 附 nginx 配置表

##### nginx.conf
```
stream {
    upstream memcached_backend {
        server ${dstip:port} weight=1 max_fails=2 fail_timeout=30s;
    }
server {
        listen $port so_keepalive=on;
        proxy_connect_timeout 300s;
        proxy_timeout 300s;
        proxy_pass memcached_backend;
    }

}
```

##### 说明:

${dstip:port} 是 fate 接受转发的 ip:port 例如 192.168.0.1:9360
$port 是监听的端口，如 19360
