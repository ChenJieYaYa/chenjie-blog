# Flume

## 一、Flume是什么？

Flume是Cloudera提供的一个高可用、高可靠**分布式的海量日志采集、聚合和传输的系统**，基于**流式架构**，灵活简单，适用于**实时推送事件**，尤其是在数据流是持续的且量级很大的情况

## 二、Flume架构

### 1.简单架构图

![1665305084941](C:\Users\cloud\Desktop\我的\my\Flume\assets\1665305084941.png)

### 2.详细架构图

![1665305173438](C:\Users\cloud\Desktop\我的\my\Flume\assets\1665305173438.png)

### 3.Agent

Agent以**事件**的形式将数据从源头送至目的地，是Flume数据传输的基本单元，由3个部分组成，分别是**Source、Channel、Sink**

#### 3.1.Source

Source负责接收数据到Flume Agent的组件，可以处理各种类型、各种格式的**日志数据**，包括avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy

#### 3.2.Channel

Channel是位于Source和Sink之间的缓冲区，类似于生产者消费者模式，Event是Channel内Flume数据传输的基本单元

Channel是线程安全的，可以同时处理几个Source的写入操作和几个Sink的读取操作

Channel可基于磁盘，也可基于内存

|  Channel种类   |                             说明                             |
| :------------: | :----------------------------------------------------------: |
|  File Channel  |      基于磁盘，不会导致数据丢失，但大量的IO是系统的瓶颈      |
| Memory Channel | 基于内存，关机、重启或程序死亡、机器宕机等情况会导致数据丢失，在不需要关心数据丢失的情景下适用 |

#### 3.3.Sink

Sink不断地轮询Channel中的事件，将这些事件批量写入到存储系统或者被发送到另一个Flume Agent

Sink是完全事务性的，在从Channel批量获取事物之前，每个Sink用Channel启动一个事务，批量事件一旦成功写出到存储系统或下一个Flume Agent，Sink就利用Channel提交事务，事务一旦被提交，该Channel从自己的内部缓冲区删除事件

Sink组件目的地包括**hdfs**、**logger**、**avro**、thrift、ipc、**file**、null、HBase、solr、自定义，其中Avro Sink是扇出流(1对多)扇入流(多对1)的基础，实现多个Flume的连接



![1665306345142](C:\Users\cloud\Desktop\我的\my\Flume\assets\1665306345142.png)








​	






