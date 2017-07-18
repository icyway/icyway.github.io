---
layout: post
title: nginx之实时打印日志到kafka（一）
category: 运维
keywords: linux，nginx，lua，kafka
---

## 背景
老板和产品给开发提出需求，为优化产品服务，需要对web日志进行分析，做一些可视化、饼状、柱状的东西。
初步方案是：利用nginx的lua模块，将日志 实时打进kafka集群，开发把kafka里面的队列数据取出来，进行分析展示。

nginx+lua+kafka是我负责搭建配置调试。

我在网上看了看，不晓得为什么很少人这么做，所以导致这样的有效成功案例特别少，借鉴的地方不是很多，只能自己试着去做。

## kafka安装调试
之前没用过kafka，这次填了不少坑。反正是测试环境，为了尽快理清逻辑实现功能，建议此处用单点kafka和单点zookeeper。
kafka[下载地址](http://kafka.apache.org/downloads.html)

kafka是需要jdk（jre）环境的，此处我安装的是openjdk1.8
解压tgz文件之后不需要编译安装，只需要把目录移动到工作目录，改一下目录名字就可以，在运行服务之前改一下配置文件。

1、vim /data/kafka/config/server.properties

`broker.id=1`此处是kafka集群中，标识不同节点的id

`listeners=PLAINTEXT://ip:9092`此处要写本机ip

`zookeeper.connect=localhost:2181`指定kafka受哪些zk调度，多个用' , '隔开。

*注：* 也可以再加上host.name  port

2、vim /data/kafka/config/zookeeper.properties

因为kafka默认受zk调度，下载的包中自带 精简zookeeper，此配置文件即为zk的配置文件，单点zk不需要改什么配置，若是集群则需要在配置文件中指定所有集群节点和端口。

3、首先 开启zookeeper服务

`/data/kafka/bin/zookeeper-server-start.sh /data/kafka/config/zookeeper.properties`

4、开启 kafka服务

` /data/kafka/bin/kafka-server-start.sh /data/kafka/config/server.properties`

5、测试kafka功能
创建topic

`bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test1`

_注_： localhost 或者本机ip，后同

查看topic

`bin/kafka-topics.sh --list --zookeeper localhost:2181`

发送消息

`bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test1`

另起终端，启动consumer

`bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test1 --from-beginning`

接受发送消息，可以在nginx或者另外的节点测试。
