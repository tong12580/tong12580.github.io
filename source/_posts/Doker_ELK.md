title: Doker ELK 的安装部署使用教程
thumbnail: https://i.v2ex.co/2Zo3bPa3.png
date: 2017/10/24 18:10:00
tags: 
    - docker
    - 日志分析
    - Logstash
    - elasticsearch
categories:
    - 搜索引擎
---

## 简介

### 1. ELK是什么？
   ELK 是由Elasticsearch，Logstash 和 kibana 三个组件组成的 一种日志收集分析系统。
   
   其中：
   1. Logstash： 主要用来收集日志，并对日志进行分析，处理与储存，并将其发送给Elasticsearch
   2. Elasticsearch： 为一开源的分布式搜索引擎 ，他可以为日志添加索引，对索引进行自动分片，提供良好的restful风格接口，提供自动的搜索负载等。
   3. Kibana : 为该日志分析系统 提供web可视化的界面和可供查询编辑的各类分析视图等。

### 2.工作流程

![ELK工作流程](https://i.v2ex.co/2Zo3bPa3.png)

Logstash从各个日志源进行日志的收集工作，并根据配置信息进行过滤，发送给Elasticsearch 添加索引，再由Kibana进行展示。

###  3.选用 elk-docker 进行部署

在这里，选用 elk-docker 来进行容器的部署工作

* 首先，使用 `sudo docker pull sebp/elk`  将相关 镜像 pull 下来；为什么使用 sebp/elk 是因为，他的文档写的 是最全面，最容易理解，[文档](http://elk-docker.readthedocs.io/)中包含了很多问题的解决办法；
* 安装前置条件：
	1. Docker至少得分配3GB的内存；
	2. Elasticsearch至少需要单独2G的内存；
	3. 防火墙开放相关端口 这个可以查看[文档](http://elk-docker.readthedocs.io/)；
	4. `sudo vim /etc/sysctl.conf` 在该文件中添加 `vm.max_map_count = 262144` 然后执行命令 `sysctl -p` 查询修改是否成功
* 安装 
	1. pull下docker镜像 其镜像的配置文件位于 `/etc/logstash/conf.d/` 文件夹下，他有几个配置文件 30-output.conf 为输出过滤，02-beats-input.conf 为输入过滤等等，可进行相应的配置替换；
	2.  如果替换的话，使用docker 的 -v 命令对 `/etc/logstash/conf.d/` 下的文件进行相应的替换。建议参考logstash的配置文档，先对logstash的配置有个初步的了解和认识。初次启动时建议按下面的方式进行启动，当成功启动后，进入docker容器，并进入`/etc/logstash/conf.d/` 下查看各个配置文件，您大概就知道如何进行配置工作了。
	3.  如果不替换 输入： `docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it --name elk sebp/elk`  接着进入容器内部 输入 `docker exec -it <container-name> /bin/bash` ，并执行命令：`/opt/logstash/bin/logstash -e 'input { stdin { } } output { elasticsearch { hosts => ["localhost"] } }'` 注意：如果看到这样的报错信息 Logstash could not be started because there is already another instance using the configured data directory.  If you wish to run multiple instances, you must change the "path.data" setting. 请执行命令：service logstash stop 然后在执行就可以了。当命令成功被执行后，看到：Successfully started Logstash API endpoint {:port=>9600} 信息后，输入：this is a dummy entry 然后回车，模拟一条日志进行测试。
	4.  打开浏览器，输入：http://ip:5601 点击创建
![创建方法](http://images2015.cnblogs.com/blog/1154245/201705/1154245-20170513171552754-2145153497.png)

如果你 Logstash 做了相应的 配置请在  `Index name or pattern` 中选对您的相应配置项 类似如下图中 我在 
30-output.conf  中做了输出的数据时间头部的修改，所以就是如下图所示：

![自我所示](http://img.blog.csdn.net/20171024173114928?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSjNva2Vy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


