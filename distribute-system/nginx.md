# Nginx

![](<../.gitbook/assets/image (143).png>)

### 模块分类：

* 核心模块：HTTP模块、EVENT模块和MAIL模块
* 基础模块：HTTP Access模块、HTTP FastCGI模块、HTTP Proxy模块和HTTP Rewrite模块，
* 第三方模块：HTTP Upstream Request Hash模块、Notice模块和HTTP Access Key模块。

### 功能分类：

* Handlers（处理器模块）。此类模块直接处理请求，并进行输出内容和修改headers信息等操作。Handlers处理器模块一般只能有一个。
* Filters （过滤器模块）。此类模块主要对其他处理器模块输出的内容进行修改操作，最后由Nginx输出。
* Proxies （代理类模块）。此类模块是Nginx的HTTP Upstream之类的模块，这些模块主要与后端一些服务比如FastCGI等进行交互，实现服务代理和负载均衡等功能。



![](<../.gitbook/assets/image (41).png>)



## Features

* 反向代理，
* 负载均衡，
* 动静分离，
* 高可用集群/高并发
* _热部署（不停掉server情况下重启）_

__

## 使用

核心就是对config file的改写，

nginx folder 可以存放静态资源，相当于静态获取，节省访问server次数

```bash
#Nginx.conf: (etc/nginx/nginx.conf)
    global块：Workprocesses: 越大，并发量越大
    Event块: 服务器与用户网络部分
    http块：使用最多的地方
        server块：虚拟主机有关， port, hostname
            location块：root & index 
```

#### 高可用性：

两台nginx: master + backup

当主服务器宕机， backup成为master Keepalived —> routing, 需要检测nginx heartbeat， 

nginx需要对外提供virtual ip

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FJtfyAUxHqOk9QWaKRPmW%2Ffile.png?alt=media)

## Installation

[For linux](http://nginx.org/en/linux_packages.html#RHEL-CentOS)

```bash
Systemctl start nginx
Systemctl status nginx
Ps -ef | grep nginx

#Default path: etc/nginx/config.d/default.conf
#Server doc: usr/share/nginx/html

firewall-cmd --state
systemctl stop firewalld
#关闭firewall才能access到
firewall-cmd --list-all
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd -reload

#/usr/local/nginx/sbin 在这个path下进行命令行操作
./nginx -v
./nginx -s stop
```





## Eg config file

```bash

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
##################################################################### www.gmall.com
    server{
        listen   80;
        server_name   www.gmall.com;
        location / {
            root gmall_www;
            index index.htm;
        }
    }

##################################################################### www.gware.com 订单管理系统
    upstream www.gware.com {
       server 127.0.0.1:9001;
    }

    server{
        listen   80;
        server_name www.gware.com;
        location / {
            proxy_pass http://www.gware.com;
        }
    }


##################################################################### payment.gmall.com
    upstream payment.gmall.com {
       server 127.0.0.1:8088;
    }

    server{
        listen   80;
        server_name payment.gmall.com;
        location / {
            proxy_pass http://payment.gmall.com;
            proxy_set_header X-forwarded-for $proxy_add_x_forwarded_for;

        }

        #正则匹配优先级高
        location ~*.(css|png|ttf|eot|svg|woff|jpg)$ {
            root gmall_payment;
        }

    }

##################################################################### order.gmall.com
    upstream order.gmall.com {
       server 127.0.0.1:8089;
    }

    server{
        listen   80;
        server_name order.gmall.com;
        location / {
            #所有资源都在tomcat, 因此不需要动静分离，dependson
            proxy_pass http://order.gmall.com;
            proxy_set_header X-forwarded-for $proxy_add_x_forwarded_for;

        }

        #正则匹配优先级高
        location ~*.(css|png|ttf|eot|svg|woff|jpg)$ {
            root gmall_order;
        }

    }



##################################################################### cart.gmall.com
    upstream cart.gmall.com {
       server 127.0.0.1:8086;
    }

    server{
        listen   80;
        server_name cart.gmall.com;
        location / {
            #所有资源都在tomcat, 因此不需要动静分离，dependson
            proxy_pass http://cart.gmall.com;
            proxy_set_header X-forwarded-for $proxy_add_x_forwarded_for;

        }

        #正则匹配优先级高
        location ~*.(css|png|ttf|eot|svg|woff|jpg)$ {
            root gmall_cart;
        }

    }


##################################################################### passport.gmall.com
    upstream passport.gmall.com {
       server 127.0.0.1:8087;
    }

    server{
        listen   80;
        server_name passport.gmall.com;
        location / {
            #所有资源都在tomcat, 因此不需要动静分离，dependson
            proxy_pass http://passport.gmall.com;
            proxy_set_header X-forwarded-for $proxy_add_x_forwarded_for;
        }

        #正则匹配优先级高
        location ~*.(css|png|ttf|eot|svg|woff|jpg)$ {
            root gmall_passport;
        }

    }


##################################################################### item.gmall.com
    upstream item.gmall.com {
       server 127.0.0.1:8083;
    }

    server{
        listen   80;
        server_name item.gmall.com;
        location / {
            #所有资源都在tomcat, 因此不需要动静分离，dependson
            proxy_pass http://item.gmall.com;
            proxy_set_header X-forwarded-for $proxy_add_x_forwarded_for;

        }

        #正则匹配优先级高
        location ~*.(css|png|ttf|eot|svg|woff|jpg)$ {
            root gmall_item;
        }

    }

##################################################################### manage.gmall.com
    upstream manage.gmall.com {
       server 127.0.0.1:8082;
    }

    server {
        listen       80;
        server_name  manage.gmall.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;


        # 首页
        location / {
            root   gmall_manage;
            index  index.html index.htm;
        } 
        
        # 静态
        location ~*.(html|js|ico|css|png|jpg|ttf)$ {
           root gmall_manage;
        } 

        # 动态 
        location ~[a-z]+ {
            proxy_pass http://manage.gmall.com;
        }
```
