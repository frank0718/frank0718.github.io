---
layout: post
title: "nginx best practice"
description: ""
category: 
tags: []
---
{% include JB/setup %}


++是否在工作中遇到过各种的nginx错误,导致服务不可用? 未注意到配置文件的细节问题,导致配置不生效? 遇到log中的提示,无法快速定位问题? 本文旨在提供一些方法或者思路,希望为大家解决各自的问题提供一些帮助.++

##(1)debug
工欲善其事,必先利其器.
测试阶段找不到问题的原因时,开启error_log debug的开关,然后去看error_log的内容,你会收获很多.
```
error_log  /path/to/log debug;
```

##(2) Nginx Error 413 Request Entity Too Large

由于
```
client_max_body_size 值默认是1m;
```
所以在一些需要上传大文件或者请求的数据量比较大的场景下,需要适当的调整这个值.

##(3)502/504错误

error_log一般会显示upstream timed out (110: Connection timed out)

```
2014/08/14 10:23:15 [error] 19210#0: *104643398 upstream timed out (110: Connection timed out) while reading response header from upstream,
```
说明本层代理是ok的,要去找upstream指向的ip的http服务是否可用;http服务可用但是有ip或者port的限制,使用telnet ip port辅助查看结果

##(4) 日志记录中HTTP状态码出现499错误
默认情况下
```
proxy_ignore_client_abort off，
```
在请求过程中如果client主动关闭连接或者客户端网络断掉，那么 Nginx 会记录 499
如果使用了
```
proxy_ignore_client_abort on 
```
不主动关闭客户端的连接;那么客户端主动断掉连接之后，Nginx 会等待后端处理完(或者超时)，然后记录后端返回的信息到日志。所以，如果后端返回 200， 就记录200 ；如果后端放回5XX ，那么就记录 5XX .如果超时(默认60s，可以用 proxy_read_timeout 设置)，Nginx 会主动断开连接，记录 504。

##(5) Nginx 403 error: directory index of [folder] is forbidden
Error_log 如下
```
2014/08/21 11:59:26 [error] 4229#0: *38 directory index of "/home/xxxx" is forbidden, client: 127.0.0.1, server: abc.com, request: "HEAD / HTTP/1.1", host: "abc.com"
```
原因是访问这个域名的首页时由于没有配置首页的rewrite,nginx默认会去找目录中的index.html,所以解决方法有
```
a)	指定首页对应的文件如,rewrite ^/*$ /somefile break;
b)	在目录中建立index.html文件
c)	配置autoindex on;这样做用户在访问首页的时候会列出这个目录下的所有文件,强烈建议不要这么做,除非有特殊需求.
```



##(6)Permission denied
Error_log如下
```
2014/09/03 12:53:49 [error] 22402#0: *11399428976 open()  failed (13: Permission denied), client: 
```
由于nginx默认用户是nobody(强烈建议不要用root用户);文件的权限如下
-rwxr-x--- 1 root root 160 Sep 3 12:44 allphoto.xml
Nobody没有read的权限,所以无法打开,当然就无法访问.改成644就可以了
-rw-r--r-- 1 root root 160 Sep 3 12:44 allphoto.xml

##(7) rewrite 死循环
Error_log如下
```
2014/09/10 11:02:04 [error] 17989#0: *6 rewrite or internal redirection cycle while processing "/test.html", client: 127.0.0.1, server: abc.com, request: "HEAD / HTTP/1.1", host: "abc.com"
```
提示rewrite或者内部重定向出现循环.解决办法:找到/test.html对应的配置,最后加上last或者break

##(8) proxy_pass 注意点

proxy_pass” cannot have URI part in location given by regular expression, or inside named location, or inside “if” statement, or inside “limit_except” block …
eg:
```
location ~* \.(jsp|do)${
index index.jsp;
proxy_pass http://localhost:8080/shop_goods;
proxy_set_header X-Real-IP $remote_addr;
}
```
当location中配置的有正则表达式,或在命名location中,或在if语句,或在limit_except 块中,proxy_pass 不能有URI的部分,只能是proxy_pass http://ip:port的形式.

##(9) nginx配置chunked开关
有时候用curl –I 可以看到reponse的header里面会有
`Transfer-Encoding:chunked`
这一项,它表明采用chunked编码方式来进行报文体的传输。chunked编码是HTTP/1.1 RFC里定义的一种编码方式。
```
chunked编码的基本原理是将大块数据分解成多块小数据，每块都可以自指定长度。这样就能更快的让页面呈现出来,数据被截成许多小片段，浏览器分段解析。但是在一些场景中,要避免分块传输,client希望请求的数据一次性传输完毕,就需要设置
```
`chunked_transfer_encoding off;`

##(10)关于正则
nginx location/if 中的正则表达式子模式(subpatterns)与逆向引用(Back references)
比如
```
location ~ ^/somedir/((abc)|(.*))/$ 
```
那么$1=abc.*,$2=abc,$3=.*,它们的序号是由左括号的顺序决定的。
	

