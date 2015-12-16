---
layout: post
title: "fabric best practice"
description: ""
category: 
tags: []
---
{% include JB/setup %}

{% inlude assets/themes/bootstrap-3/css# vi pygments.css %}

```c
/* hello world demo */
#include <stdio.h>
int main(int argc, char **argv)
{
    printf("Hello, World!\n");
    return 0;
}
```

#Fabric--运维自动化利器
##(一)介绍
fabric是一个开源的,使用python作为源码的自动化运维工具.它具有以下优点:
1. 无需server端配置,只需client和server有ssh连接.
2. 相对于saltstack,ansible更轻量级.使用安装方便.
3. 配置文件使用python语言,源码为python,运维工程师操作起来学习成本较小.出现问题可直接翻阅源码.
4. 可自定义python函数,完成自定的自动化功能.相对于ansible的模块化,显得比较自由.
...

##(二)安装
1. 推荐 pip install fabric
2. 不推荐 sudo apt-get install fabric  版本会比较旧

##(三)使用

安装完之后会有一个fab 命令来执行,默认使用pwd下面的fabfile.py 作为配置文件.也可以使用 fab -f somefile.py 来作为配置文件.具体的命令行参数可以通过
fab -h来查看

工具的核心是其配置文件,我默认就使用fabfile.py作为文件名.下面就里面一些配置做下简要介绍.

###(1)重要api
```
from fabric.api import  * 导入fabric包中的核心api函数.大概包括以下


"""
abort          fastprint      lcd            parallel       put            remote_tunnel  runs_once      show           warn_only      
cd             get            local          path           puts           require        serial         sudo           with_settings  
env            hide           open_shell     prefix         quiet          roles          settings       task           
execute        hosts          output         prompt         reboot         run            shell_env      warn   
open_shell
"""

```

其中比较重要和常用的有
```
lcd:切换本地目录
parallel:多个server并行执行任务
put:上传本地文件到server
cd:切换server的目录
get:下载server的文件到本地
local:执行本地的命令
sudo:在server上执行sudo命令
env:设置本地的环境变量.
roles:选定一组role
execute:绑定多个方法
hosts:选定一组server
run:执行具体的任务.

```


###(2)输出着色
from fabric.colors import * 导入fabric包中的颜色函数,用来对输出进行着色.

###(3)设定环境变量
```
env.user = 'bjzhangfeng'#指定remote server中的用户
env.port  = '22' #指定remote server ssh的port
这个设置是全局的,在具体的函数里面可以对其重新设定来覆盖这个全局设置,比如某一台server 的ssh port 不为22.

env.roledefs = {  #设定一组角色, 比如abc 代表一组server,xyz代表一组server
    'abc':[
    '1.1.1.1',
    '1.1.1.2',
    '1.1.1.3',
    ],
  'xyz':[
    '2.1.1.1',
    '2.1.1.2',
    '2.1.1.3',
    ],
}
```


###(4)定义任务
```
(a):run方法
def w():
    run('w')
通过定义函数来执行具体的任务,run指在server上 以 env.user用户执行w命令.
fab -R abc w
在abc这组server上执行w
fab -H 1.1.1.1 w
在单台server上执行 w
fab -H 1.1.1.1,1.1.1.2 w
在这两台机器上执行w

(b)local 方法
def w():
   local('w')
fab w
在本地执行w命令

(c)sudo 方法
def ss():
  sudo('ss -s')
fab -R abc ss
在abc组以sudo的方式执行ss -s

(d)roles/hosts装饰器
@roles('abc')
def w():
    run('w')

fab w
绑定w方法到abc服务器角色. 对于某个方法只适用于某个role的情形下,非常好用.

(e)parallel装饰器.
@parallel
@roles('abc')
def w():
    run('w')
fab w
以并行的方式在abc一组server上,执行w命令.由于fabric默认是串行的执行任务.在server比较多的情况下,并行执行可以节省时间.

(f)put方法/get 方法
def put():
   put('~/locafile','~/')
fab -R abc put
把本地文件put到server端

(g) 对方法传参数,
def curlc(url):
    while True:
        local('curl -I %s ; sleep 1' % (url,))
fab curlc:http://www.163.com
对于多个参数用逗号分隔
def curl(url,ip='2.2.2.2'):
    while True:
        local('curl -I %s -x %s:80; sleep 1' % (url,ip))
fab curl:http://www.163.com,1.1.1.1

(h)execute方法
def ab(url):
    execute(a,url)
    execute(b,url)
fab -R abc ab
当需要合并执行多个方法的时候可以用这种方式,

(i)更换user,port
def whoami():
    env.user = 'fabric'
    run('whoami')
fab -R xyz whoami
在xyz上以fabric用户执行

 def ew():
    env.port = '8822'
    run('w')
fab -R xyz w
使用8822端口登陆ssh执行命令

```


##(四)总结
本文简单介绍了fabric工具,并展示了一些基本的使用方法.有需求的同学可以深入研究,以提高工作效率.

##(五)参考文档
http://docs.fabfile.org/en/1.10/index.html
