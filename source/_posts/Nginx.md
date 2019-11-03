title: 'Nginx '
author: Sunkz
tags:
  - nginx

photos:

- https://s2.ax1x.com/2019/11/03/KX7DTe.png

categories: []
date: 2019-11-03 17:34:00

---
### Nginx-Conf

```nginx
#正向代理
server {
	listen 8887;
	resolver 8.8.8.8;	
	
	location / {
		proxy_pass http://$http_host$request_uri;
	}
}
```

```nginx
#转发
server {
	listen 8888;
	server_name localhost;

	location / {
		proxy_pass http://192.168.0.116:8080/;
	}	
}
```

```nginx
#重写
server {
    listen 8080;
    server_name localhost;
    rewrite ^/git/(.*) https://github.com/sunkz permanent;
}
```

```nginx
#网关跨域
server {
	listen 80;
	server_name localhsot;

	location /8081/A {
		proxy_pass http://192.168.0.116:8081/A;
	}

	location /8082/B {
		proxy_pass http://192.168.0.116:8082/B;
	}

}
```

```nginx
#负载均衡
upstream lb{
	server 192.168.0.116:8081;
	server 192.168.0.116:8082;
}

server {
	listen 80;
	server_name localhsot;

	location / {
		proxy_pass http://lb/;
    include proxy_params;
	}
}

#proxy_params
proxy_connect_timeout 1;
proxy_send_timeout 1;
proxy_read_timeout 1;

proxy_redirect default;

proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;

proxy_connect_timeout 30;
proxy_send_timeout 60;
proxy_read_timeout 60;

proxy_buffer_size 32k;
proxy_buffering on;
proxy_buffers 4 128k;
proxy_busy_buffers_size 256k;
proxy_max_temp_file_size 256k;
```

```nginx
#静态资源代理
#防DDOS
limit_req_zone $binary_remote_addr zone=one:10m rate=50r/m
server {
    limit_req zone=one;
    
    listen       80;
    server_name  localhost;
	
    location ~ .*\.(jpg|gif|png)$ {
        gzip on;
        gzip_http_version 1.1;
        gzip_comp_level 2;
        gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        root  /opt/app/code/images;
    }

    location ~ .*\.(txt|xml)$ {
        gzip on;
        gzip_http_version 1.1;
        gzip_comp_level 1;
        gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        root  /opt/app/code/doc;
    }

    location ~ ^/download {
        gzip_static on;
        tcp_nopush on;
        root /opt/app/code;
    }
    
    location ~ .*\.(htm|html)$ {
        add_header Access-Control-Allow-Origin *; 
        add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
        expires 24h;
        root  /opt/app/code;
    }
	
	#防盗链
    location ~* \.(gif|jpg|png|swf|flv)$ {
    	valid_referers none blocked www.skz.com;
        if ($invalid_referer) {
            return 403;
        }
    }

}
```

### IP_HASH的负载均衡

```shell
#最简单的hash函数
#增加或移除Server,需要改变hash函数
#会影响到大多数Server,Client-Server原有会话不能保持
h = hash(ip) % 3
```

```shell
#一致性hash函数
#首先对Server进行一次hash,所有Server看成一个圆环
#Client请求过来计算hash,将其推送到顺时针行走碰到的第一个Server上
#增加或者移除一个Server,仅会对其前一个Server产生影响
h = hash(ip) % 2^32
```

### [ketama](http://www.last.fm/user/RJ/journal/2007/04/10/rz_libketama_-_a_consistent_hashing_algo_for_memcache_clients)

[Nginx Reference](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)

> Generic [**Hash**](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#hash) – The server to which a request is sent is determined from a user‑defined key which can be a text string, variable, or a combination. For example, the key may be a paired source IP address and port, or a URI as in this example:
>
> ```
> upstream backend {
>  thash $request_uri consistent;
>  server backend1.example.com;
>  server backend2.example.com;
> }
> ```
>
> The optional [`consistent`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#hash) parameter to the `hash` directive enables [ketama](http://www.last.fm/user/RJ/journal/2007/04/10/rz_libketama_-_a_consistent_hashing_algo_for_memcache_clients) consistent‑hash load balancing. Requests are evenly distributed across all upstream servers based on the user‑defined hashed key value. If an upstream server is added to or removed from an upstream group, only a few keys are remapped which minimizes cache misses in the case of load‑balancing cache servers or other applications that accumulate state.

### 扩展

[How Nginx Process A Request](https://juejin.im/post/5bf671a4e51d4546d60a9bd1)

[Load Balance](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)

[Self Sign](http://www.ruanyifeng.com/blog/2018/02/nginx-docker.html)

[Forward Proxy](https://www.jianshu.com/p/ab2be9d6040f)


