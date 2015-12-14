---
layout: post
title: "Github blog"
description: ""
category: 
tags: []
---
{% include JB/setup %}

# 搭建github博客 

**在国内的环境装软件真是TM的艰辛**

先列出环境 Ubuntu12.04 i386

##(1) 安装rvm 
按照google搜出来的教程.
curl -L https://get.rvm.io | bash -s stable 根本没反应好不好,
只好去vps上把https://get.rvm.io代码拿到 push到git 上,或者copy到本地. 执行
bash rvm.sh stable 

##(2) 保证github 或者bitbucket 可用.
rvm.sh 要从github 或者 bitbucket.org取代码.

##(3)配置 /etc/profile 让系统找到rvm的位置.
rvm安装完成后, 
默认rvm 装在 /usr/local/rvm, 
编辑/etc/profile, 最后加入
export PATH=/usr/local/../bin:$PATH
安装完成 
rvm -v
rvm 1.26.11

##(4)配置ubuntu的源
使用souhu的12.04的源. 
apt-get update  , 
因为默认的源 没法更新libsqlite ,  坑爹.  使用搜狐的就可以了.

rvm requirements 查看依赖,并自动解决依赖.
执行之后下面的都装上了,为安装ruby做准备,又离成功近了唉
apt-get --no-install-recommends --yes install gawk libyaml-dev libsqlite3-dev sqlite3 libgdbm-dev libtool bison

##(5)第一次安装ruby (fail)
rvm install 2.0.0
No binary rubies available for: ubuntu/12.04/i386/ruby-2.0.0-p643.
还要编译安装.
....中间省略
Installing Ruby from source to: /usr/local/rvm/rubies/ruby-2.0.0-p643, this may take a while depending on your cpu(s)...

ruby-2.0.0-p643 - #extracting ruby-2.0.0-p643 to /usr/local/rvm/src/ruby-2.0.0-p643....
ruby-2.0.0-p643 - #configuring..................................................
ruby-2.0.0-p643 - #post-configuration..
ruby-2.0.0-p643 - #compiling.............................................................................
ruby-2.0.0-p643 - #installing..............................
ruby-2.0.0-p643 - #making binaries executable..
ruby-2.0.0-p643 - #downloading rubygems-2.4.8  卡住不动了
好吧.ctrl c取消了安装过程,downloading是在走不动啊 


##(6)第一次配置ruby (fail)
rvm 2.0.0 --default
报错
Gemset '' does not exist, 'rvm ruby-2.0.0-p643 do rvm gemset create ' first, or append '--create'.
看来ruby 和gem是需要同时安装的.
具体怎么单独安装gem , 查不到...

痛定思痛. 再去google. 

##(7)第二次安装ruby (success)
由于rvm安装ruby时,使用的是国外的源, 速度不忍直视.

修改rvm的源 为taobao的 
$ sed -i -E 's!https?://cache.ruby-lang.org/pub/ruby!https://ruby.taobao.org/mirrors/ruby!'$rvm_path/config/db
这里的rvmpath就是/usr/local/rvm/目录.

rvm install 2.2.0

完成. 速度还挺快. 

当做默认的rvm版本
 rvm use 2.2.0 --default
Using /usr/local/rvm/gems/ruby-2.2.0

查询已经安装的ruby
rvm list
卸载一个已安装版本
rvm remove 2.0.0 

ruby -v
ruby 2.2.0p0 (2014-12-25 revision 49005) [i686-linux]

查看gem的版本
 gem -v
2.4.8

走到这一步,泪流满面,感谢方校长....

##(8) 配置gem源
速度问题,必须修改 gem的源 为淘宝的 . 
$ gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/
$ gem sources -l
*** CURRENT SOURCES ***

https://ruby.taobao.org
请确保只有 ruby.taobao.org

有选择的安装rails
$ gem install rails
https://ruby.taobao.org/


##(9)安装 jeklly (终于来了)

gem install jekyll -V

第一次装报ssl的错误, 查了一大堆 也没看出来咋搞. 
具体参考:
https://gist.github.com/luislavena/f064211759ee0f806c88
https://raw.githubusercontent.com/rubygems/rubygems/master/lib/rubygems/ssl_certs/AddTrustExternalCARoot-2048.pem

http://blacktha.com/2015/07/06/tech/Ruby/
http://yunkus.com/article/ziyuan/287.html

等了一会再装就ok了(网络问题?). 

jekyll -v
jekyll 3.0.1


##(10)  搭建博客
前提:
按照github的提示一步步建立自己的repo
然后参考了一个博客,使用现成的模板

git clone https://github.com/plusjade/jekyll-bootstrap.git
jekyll server
启动一个本地的server . 

创建一篇post
rake post title="Hello World"
rake page name="about.md"

然后可以把这个目录的内容cp到自己的github项目里 , 
需要注意
修改文件：_config.yml，设置production_url
~ vi _config.yml
production_url : http://frank0718.github.io

git add -all
git commit -m "add "
git  push  origin master
然后就能看到博客了 

