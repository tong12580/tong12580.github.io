title: nginx代理TCP模块
thumbnail: https://i.v2ex.co/2Zo3bPa3.png
date: 2017/06/28 09:30:00
tags:
    - tcp
    - nginx
categories:
    - nginx
---

#### nginx1.9之后的版本可代理TCP链接

   1. 其中Windows版本可直接使用
   2. Linux版本需要在编译(./configure)时添加--with-stream参数
   3. 简单 示例如下
   
1.简单代理

```
stream {
	server {
		listen 22;
		proxy_connect_timeout 20s;
		proxy_timeout 2m;
		proxy_pass ip:port;
	}
}

```

2.负载均衡 操作与Http代理非常的类似 下面是一个官方Demo

```
stream {
    upstream backend {
        hash $remote_addr consistent;
        server backend1.example.com:12345 weight=5;
        server 127.0.0.1:12345 max_fails=3 fail_timeout=30s;
        server unix:/tmp/backend3;
    }
 
    server {
        listen 12345;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass backend;
    }
 
    server {
        listen [::1]:12345;
        proxy_pass unix:/tmp/stream.socket;
    }
}

```

完