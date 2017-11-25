title: 服务器建立redis服务傻瓜教程
date: 2016/06/02 10:39:00
tags:
    - redis
categories:
    - redis
---

#服务器建立redis服务攻略
* 首先，我是用的是Ubantu的service,
* 下载并解压redis tar xzvf -xxxx,我是解压在了 /usr/local/redis下
* ***********************************************************
***** 接下来就是傻瓜教程，按顺序执行：
* cd /usr/local/redis/redis-2.8.19/src
* make
* make install
* sudo mkdir -p /usr/local/redis/bin
* sudo mkdir -p /usr/local/redis/etc
* [neil@neilhost src]$ cd ..
* sudo mv ./redis.conf /usr/local/redis/etc/
* cd src

```
sudo mv mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-dump redis-cli redis-sentinel redis-server /usr/local/redis/bin/
```

* 制作redis服务 cd /usr/local/redis/redis-2.8.19/utils   cp redis_init_script  /etc/init.d/redis
* vim /etc/init.d/redis
* 原文件第2行  添加 #chkconfig: 2345 80 90   第20行 $EXEC $CONF 加 &
* 第10行 PIDFILE=/var/run/redis_${REDISPORT}.pid   -> PIDFILE=/var/run/redis.pid
* mkdir /etc/redis
* cd -
* cp ./redis.conf /etc/redis/6379.conf
* sudo apt-get install chkconfig
* chkconfig --add redis(添加服务，确认 chkconfig是否安装)
* $sudo ln -s /usr/lib/insserv/insserv /sbin/insserv
* service redis start
* vi /etc/profile
* 在最后一行添加 export PATH="$PATH:/usr/local/redis/bin"
* source /etc/profile
* 即可使用 $ redis-cli service redis
*********************************
#### 后记
##### 
在Ubuntu下安装service服务，可能会报如下错误：
 /sbin/insserv: No such file or directory

据说这是Ubuntu的小bug，
$sudo ln -s /usr/lib/insserv/insserv /sbin/insserv