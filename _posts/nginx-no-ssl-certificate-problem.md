---
layout: post
title: Nginx 配置 Https 出现no "ssl_certificate" is defined问题原因及解决方法
categories: [Nginx]
description: Nginx 配置 Https 出现no "ssl_certificate" is defined问题原因及解决方法
keywords: Nginx, 运维
---
今天为线上Nginx更换SSL证书的过程中遇到了一个诡异的错误，折腾了挺长时间，事后回想起来还是挺容易跳坑的，所以想记录下来，
一是留作以后查看，二来也许可以帮助一帮跳坑的小伙伴。  
### 现象
"配置好"（当然只是自认为配置好）nginx ssl证书后，请求http站点正常，请求https站点打不开，nginx错误日志如下：
```log
2017/05/02 16:19:12 [error] 19656#19656: *4 no "ssl_certificate" is defined in server listening on SSL port while SSL handshaking, client: xxx, server: 0.0.0.0:443
2017/05/02 16:19:12 [error] 19656#19656: *5 no "ssl_certificate" is defined in server listening on SSL port while SSL handshaking, client: xxx, server: 0.0.0.0:443
2017/05/02 16:19:12 [error] 19656#19656: *6 no "ssl_certificate" is defined in server listening on SSL port while SSL handshaking, client: xxx, server: 0.0.0.0:443
```
看错误提示意思是ssl_certificate没有配置，可是检查nginx配置，ssl_certificate和ssl_certificate_key两个选项明明都已经配置了啊，
再检查配置的证书路径、权限也都没有任何问题，真是百思不得其解啊。  
最后在网上搜了一通，终于在StackOverflow一个问题找到了答案（问题地址懒得找了，就不贴了哈），大概意思是说ssl_certificate必须在http段中先定义，
在server段才配置ssl_certificate已经来不及了，检查我的nginx配置，ssl_certificate确实只在server段定义，而在http段未定义，nginx配置如下:
```nginx
server {
    listen       443 ssl;
    server_name  x.example.com;

    ssl_certificate /etc/certs/example.com/1_example.com_bundle.crt;
    ssl_certificate_key /etc/certs/example.com/2_example.com.key;

    #charset koi8-r;
    access_log  /home/data/logs/nginx/x.example.access.log  main;
    error_log  /home/data/logs/nginx/x.example.error.log  error;

    location / {
        proxy_pass   http://127.0.0.1:9001;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}
```
### 解决方案
既然有人提出原因，那就试一试呗，把上面两行ssl_certificate的配置移到http段配置内，reload nginx，果然解决了。纳尼？解决了？竟然就这么一把解决了！！弱鸡啊！
### 事后分析
* 为什么我会要把ssl_certificate放到server段？  
    因为申请的证书是每个子域名的证书，所以并不是全站试用，所以感觉没必要放到http中作为全局配置，只在子域名的单独配置文件中声明了。
* 既然这样，ssl_certificate还有必要写到server段吗？  
    答案是肯定的，比如我的情况就必须在server段写ssl_certificate，因为我几个子域名的SSL证书都不同，必须在每个server不同的域名下指定不同的证书。
    而且事实证明，虽然在http声明了全局的SSL证书，但是可以再不同的server段为不同子域名再重写SSL证书路径，所以在我的配置中，
    http中声明的全局SSL证书只是个打酱油的，只是为了不让nginx报上述no "ssl_certificate" is defined的错误，真正生效的证书配置还是在server段。
