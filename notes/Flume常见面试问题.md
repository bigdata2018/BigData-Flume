# Flume企业面试题

<nav>
<a href="#一、你是如何实现Flume数据传输的监控的">一、你是如何实现Flume数据传输的监控的</a><br/>
<a href="#二、Flume的Source，Sink，Channel的作用？你们Source是什么类型？">二、Flume的Source，Sink，Channel的作用？你们Source是什么类型？</a><br/>
<a href="#三、Flume的Channel Selectors">三、Flume的Channel Selectors</a><br/>
<a href="#四、Flume参数调优">四、Flume参数调优</a><br/>
    <a href="#五、Flume的事务机制">五、Flume的事务机制</a><br/>
    <a href="#六、Flume采集数据会丢失吗?">六、Flume采集数据会丢失吗?</a><br/>
    </nav>







### 一、你是如何实现Flume数据传输的监控的

使用第三方框架Ganglia实时监控Flume



### 二、Flume的Source，Sink，Channel的作用？你们Source是什么类型？

**1.作用**

（1）Source组件是专门用来收集数据的，可以处理各种类型、各种格式的日志数据，包括avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy

（2）Channel组件对采集到的数据进行缓存，可以存放在Memory或File中。

（3）Sink组件是用于把数据发送到目的地的组件，目的地包括Hdfs、Logger、avro、thrift、ipc、file、Hbase、solr、自定义。

**2.我公司采用的Source类型为：**

（1）监控后台日志：exec

（2）监控后台产生日志的端口：netcat

Exec spooldir



### 三、Flume的Channel Selectors

![img](file:///C:/Users/whj/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)



### 四、Flume参数调优

**1.Source**

增加Source个（使用Tair Dir Source时可增加FileGroups个数）可以增大Source的读取数据的能力。例如：当某一个目录产生的文件过多时需要将这个文件目录拆分成多个文件目录，同时配置好多个Source 以保证Source有足够的能力获取到新产生的数据。

batchSize参数决定Source一次批量运输到Channel的event条数，适当调大这个参数可以提高Source搬运Event到Channel时的性能。

**2.Channel** 

type 选择memory时Channel的性能最好，但是如果Flume进程意外挂掉可能会丢失数据。type选择file时Channel的容错性更好，但是性能上会比memory channel差。

使用file Channel时dataDirs配置多个不同盘下的目录可以提高性能。

Capacity 参数决定Channel可容纳最大的event条数。transactionCapacity 参数决定每次Source往channel里面写的最大event条数和每次Sink从channel里面读的最大event条数。**transactionCapacity需要大于Source和Sink的batchSize参数。**

**3.Sink** 

增加Sink的个数可以增加Sink消费event的能力。Sink也不是越多越好够用就行，过多的Sink会占用系统资源，造成系统资源不必要的浪费。

batchSize参数决定Sink一次批量从Channel读取的event条数，适当调大这个参数可以提高Sink从Channel搬出event的性能。



### 五、Flume的事务机制

Flume的事务机制（类似数据库的事务机制）：Flume使用两个独立的事务分别负责从Soucrce到Channel，以及从Channel到Sink的事件传递。比如spooling directory source 为文件的每一行创建一个事件，一旦事务中所有的事件全部传递到Channel且提交成功，那么Soucrce就将该文件标记为完成。同理，事务以类似的方式处理从Channel到Sink的传递过程，如果因为某种原因使得事件无法记录，那么事务将会回滚。且所有的事件都会保持到Channel中，等待重新传递。



### 六、Flume采集数据会丢失吗?

根据Flume的架构原理，Flume是不可能丢失数据的，其内部有完善的事务机制，Source到Channel是事务性的，Channel到Sink是事务性的，因此这两个环节不会出现数据的丢失，唯一可能丢失数据的情况是Channel采用memoryChannel，agent宕机导致数据丢失，或者Channel存储数据已满，导致Source不再写入，未写入的数据丢失。

Flume不会丢失数据，但是有可能造成数据的重复，例如数据已经成功由Sink发出，但是没有接收到响应，Sink会再次发送数据，此时可能会导致数据的重复。