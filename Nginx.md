# Nginx安装

## Nginx安装并配置ssl

第一步：确认已经安装必要依赖库

```shell
yum install gcc-c++  
yum install pcre pcre-devel  
yum install zlib zlib-devel  
yum install openssl openssl--devel 
```

第二步：从[官网](http://nginx.org/en/download.html)下载最新版本的nginx并解压

```shell
wget http://nginx.org/download/nginx-1.18.0.tar.gz

tar -zxvf nginx-1.18.tar
```

第三步：确认安装open-ssl

```shell
yum -y install openssl openssl-devel
```

第四步：在解压的文件夹内`nginx-1.18`执行并安装，nginx会被安装到`/usr/local/nginx`

```shell
./configure --with-http_ssl_module

make

make install
```

## https配置文件

```
#user  nobody;
worker_processes  3;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  coldcoder.com;
	rewrite ^(.*)$  https://$host$1 permanent;
	} 




    # HTTPS server

server {
 listen 443 ssl;
 server_name coldcoder.com;
 ssl on;
 root html;
 index index.html index.php index.htm;
 ssl_certificate   cert/coldcoder.com.pem;
 ssl_certificate_key  cert/coldcoder.com.key;
 ssl_session_timeout 2m;
 ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
 ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
 ssl_prefer_server_ciphers on;
 #location / {
 #    root html;
 #    index index.php index.html index.htm index.jsp;
 #        proxy_pass http://127.0.0.1:8080;
 #	 add_header Access-Control-Allow-Origin *;
 #	 }

location /taoquan{
	proxy_pass http://47.92.2.46:9090;
	add_header Access-Control-Allow-Origin *;
	}
location /zmlh{
	proxy_pass http://47.92.2.46:9091;
	add_header Access-Control-Allow-Origin *;
  	}
}
}

```



# Nginx常用命令

在`/usr/local/nginx/bin`下使用命令

查看版本号：`./nginx -v`

启动：`./nginx`

关闭：`./nginx -s stop`

重新加载配置文件：`./nginx -s reload`

# 配置文件nginx.conf

`nginx.conf`文件内容：

```shell
####################全局块
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

####################events块
events {
    worker_connections  1024;
}

####################http块
http {
    include       mime.types;
    default_type  application/octet-stream;
    client_max_body_size 1024m;
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       81;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

 server {
        listen       9001;
        server_name  localhost;

        location ~ /eduservice/ {
            proxy_pass http://localhost:8001;
        }

        location ~ /eduoss/ {
            proxy_pass http://localhost:8002;
        }

        location ~ /eduvod/ {
            proxy_pass http://localhost:8003;
        }

        location ~ /cmsservice/ {
            proxy_pass http://localhost:8004;
        }
    }
}

```



##配置文件由3部分组成：

###第一部分：全局块

主要会设置一些影响nginx服务器整体运行的配置指令

```shell
#user  nobody;
worker_processes  1; 	#处理并发服务的关键配置

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;
```

###第二部分：events块

影响用户与Nginx服务器的网络连接

```shell
events {
    worker_connections  1024;	#支持的最大连接数
}
```

###第三部分：http块

http块又包括http全局块和server块

####http全局块 

```shell

```

#### server块

每个http块可以包含多个server块，每个server块就相当于一个虚拟主机。

每个server块也分为全局server块，以及可以包含多个location块。

**全局server块**

最常见的配置是本虚拟机主机的监听配置和本虚拟机主机的名称或IP配置

**location块**

主要是基于nginx服务器接收到的请求字符串，对虚拟主机名称之外的字符串进行匹配，对特定的请求进行处理。

```shell
 server {
        listen       81;	#目前监听的端口
        server_name  localhost; #配置基于名称的虚拟主机

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }
        }
```

# 反向代理

##一

实现效果

输入`www.coldcoder.com `进入tomcat首页

分析：

输入`www.coldcoder.com `后服务器默认监听的是80端口

```shell
server {
        listen       80;	
        server_name  coldcoder.cn;
        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            proxy_pass	http://127.0.0.1:8080
            index  index.html index.htm;
        }
        }
```



## location中正则详解

```shell
location [ = | ~ | ~* | ^~] uri { ... }
```

- `=`：用于不含正则表达式的uri前，要求请求字符串与uri严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求
- `~`：用于表示uri包含正则表达式并且**区分**大小写
- `~*`：用于表示uri包含正则表达式，并且**不区分**大小写
- `^~`：用于不含正则表达式的uri前，要求Nginx服务器找到标志uri和请求字符串匹配度最高的location后，立即使用此location处理请求，不在使用location块中的正则uri和请求字符串做匹配。

# 负载均衡

1. 定义负载均衡服务器列表

   默认



```shell
http{
	...
	upstream myserver{		#定义负载均衡的服务列表，upstream后的名字自定义
		#ip_hash;
		#fair;
		#格式为：server ip:[port] [weight=n]
		server 100.100.100.1:8080 weight=5;
		server 100.100.100.2:8081;
	}
	
	server{
	listen	80;
	server_name:	coldcoder.cn;
	
	location / {
		root   html;
		proxy_pass http://myserver;		#使用自定义的负载均衡列表
		index  index.html index.htm;
	}
	}
}
```

分配服务器的策略：

1. 默认使用轮询方式：按请求时间顺序注意分配，如果某个服务器down掉，会被自动剔除
2. weight：代表权重，默认为1，权重越高，被分配的就越多。
3. ip_hash：直接在配置文件中加入ip_hash。每个请求安装ip的hash结果分配，这样每个访客固定一个后端服务器，可以解决session的问题
4. fair：直接在配置文件中加入fair；按后端服务器的响应时间来分配，响应时间端点额优先分配。

# 动静分离

把动态请求跟静态请求分开，可以理解为使用nginx处理静态页面，使用tomcat处理动态页面，大致有两种解决方案：

- 把静态文件独立成单独的域名，放在独立的服务器上【主流方式】
- 动态跟静态文件混在一起发布，通过nginx分开