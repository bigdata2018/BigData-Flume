# Flume入门案例

<nav>
<a href="#一、监控端口数据官方案例">一、监控端口数据官方案例</a><br/>
<a href="#二、实时监控单个追加文件">二、实时监控单个追加文件</a><br/>
<a href="#三、实时监控目录下多个新文件">三、实时监控目录下多个新文件</a><br/>
<a href="#四、实时监控目录下的多个追加文件">四、实时监控目录下的多个追加文件</a><br/>
</nav>






## 一、监控端口数据官方案例

**1.案例需求：**

```
使用Flume监听一个端口，收集该端口数据，并打印到控制台
```

**2.需求分析:**

![img](file:///C:/Users/whj/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)



**3.实现步骤**

（1）安装netcat工具

```
[nogc@hadoop102 software]$ sudo yum install -y nc
```

（2）判断44444端口是否被占用

```
[nogc@hadoop102 flume-telnet]$ sudo netstat -tunlp | grep 44444
```

（3）创建Flume Agent配置文件flume-netcat-logger.conf

在flume目录下创建job文件夹并进入job文件夹。

```
[nogc@hadoop102 flume]$ mkdir job
[nogc@hadoop102 flume]$ cd job/
```

在job文件夹下创建Flume Agent配置文件flume-netcat-logger.conf。

```
[nogc@hadoop102 job]$ vim flume-netcat-logger.conf
```

在flume-netcat-logger.conf文件中添加如下内容。

添加内容如下：

```
添加内容如下：
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

注：配置文件来源于官方手册http://flume.apache.org/FlumeUserGuide.html

![img](file:///C:/Users/whj/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

（4）先开启flume监听端口

第一种写法：

```
[nogc@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name a1 --conf-file job/flume-netcat-logger.conf -Dflume.root.logger=INFO,console
```

第二种写法：

```
[nogc@hadoop102 flume]$ bin/flume-ng agent -c conf/ -n a1 -f job/netcat-logger.conf -Dflume.root.logger=INFO,console
```

参数说明：

​    --conf/-c：表示配置文件存储在conf/目录

​    --name/-n：表示给agent起名为a1

​    --conf-file/-f：flume本次启动读取的配置文件是在job文件夹下的flume-telnet.conf文件。

​    -Dflume.root.logger=INFO,console ：-D表示flume运行时动态修改flume.root.logger参数属性值，并将控制台日志打印级别设置为INFO级别。日志级别包括:log、info、warn、error。

（5）使用netcat工具向本机的44444端口发送内容

```
[nogc@hadoop102 ~]$ nc localhost 44444
hello 
nogc
```

（6）在Flume监听页面观察接收数据情况

![img](file:///C:/Users/whj/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

思考：nc hadoop102 44444，flume能否接收到？





## 二、实时监控单个追加文件

**1.案例需求：实时监控Hive日志，并上传到HDFS中**

**2.需求分析：**

![img](file:///C:/Users/whj/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

**3.实现步骤：**

（1）Flume要想将数据输出到HDFS，依赖Hadoop相关jar包

检查/etc/profile.d/my_env.sh文件，确认Hadoop和Java环境变量配置正确

```
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_212
export PATH=$PATH:$JAVA_HOME/bin

##HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

（2）创建flume-file-hdfs.conf文件创建文件

```
[nogc@hadoop102 job]$ vim flume-file-hdfs.conf
```

注：要想读取Linux系统中的文件，就得按照Linux命令的规则执行命令。由于Hive日志在Linux系统中所以读取文件的类型选择：exec即execute执行的意思。表示执行Linux命令来读取文件。

添加如下内容

```
# Name the components on this agent
a2.sources = r2
a2.sinks = k2
a2.channels = c2

# Describe/configure the source
a2.sources.r2.type = exec
a2.sources.r2.command = tail -F /opt/module/hive/logs/hive.log
a2.sources.r2.shell = /bin/bash -c

# Describe the sink
a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://hadoop102:9820/flume/%Y%m%d/%H
#上传文件的前缀
a2.sinks.k2.hdfs.filePrefix = logs-
#是否按照时间滚动文件夹
a2.sinks.k2.hdfs.round = true
#多少时间单位创建一个新的文件夹
a2.sinks.k2.hdfs.roundValue = 1
#重新定义时间单位
a2.sinks.k2.hdfs.roundUnit = hour
#是否使用本地时间戳
a2.sinks.k2.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a2.sinks.k2.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a2.sinks.k2.hdfs.fileType = DataStream
#多久生成一个新的文件
a2.sinks.k2.hdfs.rollInterval = 60
#设置每个文件的滚动大小
a2.sinks.k2.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a2.sinks.k2.hdfs.rollCount = 0

# Use a channel which buffers events in memory
a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.r2.channels = c2
a2.sinks.k2.channel = c2
```

**注意**：

对于所有与时间相关的转义序列，Event Header中必须存在以 “timestamp”的key（除非hdfs.useLocalTimeStamp设置为true，此方法会使用TimestampInterceptor自动添加timestamp）。

```
a3.sinks.k3.hdfs.useLocalTimeStamp = true
```

![img](file:///C:/Users/whj/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

（3）运行Flume

```
[nogc@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name a2 --conf-file job/flume-file-hdfs.conf
```

（4）开启Hadoop和Hive并操作Hive产生日志

```
[nogc@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh
[nogc@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh
[nogc@hadoop102 hive]$ bin/hive
hive (default)>
```

（5）在HDFS上查看文件。

![image-20200504215826284](C:\Users\whj\AppData\Roaming\Typora\typora-user-images\image-20200504215826284.png)



## 三、实时监控目录下多个新文件

**1.案例需求：使用Flume监听整个目录的文件，并上传至HDFS**

**2.需求分析：**

![img](file:///C:/Users/whj/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

**3.实现步骤：**

（1）创建配置文件flume-dir-hdfs.conf

创建一个文件

```
[nogc@hadoop102 job]$ vim flume-dir-hdfs.conf
```

添加如下内容:

```
a3.sources = r3
a3.sinks = k3
a3.channels = c3

# Describe/configure the source
a3.sources.r3.type = spooldir
a3.sources.r3.spoolDir = /opt/module/flume/upload
a3.sources.r3.fileSuffix = .COMPLETED
a3.sources.r3.fileHeader = true
#忽略所有以.tmp结尾的文件，不上传
a3.sources.r3.ignorePattern = ([^ ]*\.tmp)

# Describe the sink
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path = hdfs://hadoop102:9820/flume/upload/%Y%m%d/%H
#上传文件的前缀
a3.sinks.k3.hdfs.filePrefix = upload-
#是否按照时间滚动文件夹
a3.sinks.k3.hdfs.round = true
#多少时间单位创建一个新的文件夹
a3.sinks.k3.hdfs.roundValue = 1
#重新定义时间单位
a3.sinks.k3.hdfs.roundUnit = hour
#是否使用本地时间戳
a3.sinks.k3.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a3.sinks.k3.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a3.sinks.k3.hdfs.fileType = DataStream
#多久生成一个新的文件
a3.sinks.k3.hdfs.rollInterval = 60
#设置每个文件的滚动大小大概是128M
a3.sinks.k3.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a3.sinks.k3.hdfs.rollCount = 0

# Use a channel which buffers events in memory
a3.channels.c3.type = memory
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.r3.channels = c3
a3.sinks.k3.channel = c3
```

![img](file:///C:/Users/whj/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

（2）启动监控文件夹命令

```
[nogc@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name a3 --conf-file job/flume-dir-hdfs.conf
```

说明：在使用Spooling Directory Source时，不要在监控目录中创建并持续修改文件；上传完成的文件会以.COMPLETED结尾；被监控文件夹每500毫秒扫描一次文件变动。

（3）向upload文件夹中添加文件

在/opt/module/flume目录下创建upload文件夹

```
[nogc@hadoop102 flume]$ mkdir upload
```

向upload文件夹中添加文件

```
[nogc@hadoop102 upload]$ touch nogc.txt
[nogc@hadoop102 upload]$ touch nogc.tmp
[nogc@hadoop102 upload]$ touch nogc.log
```

（4）查看HDFS上的数据

![image-20200504220140272](C:\Users\whj\AppData\Roaming\Typora\typora-user-images\image-20200504220140272.png)

（5）等待1s，再次查询upload文件夹

```
[nogc@hadoop102 upload]$ ll
总用量 0
-rw-rw-r--. 1 nogc nogc 0 5月  20 22:31 nogc.log.COMPLETED
-rw-rw-r--. 1 nogc nogc 0 5月  20 22:31 nogc.tmp
-rw-rw-r--. 1 nogc nogc 0 5月  20 22:31 nogc.txt.COMPLETED
```



## 四、实时监控目录下的多个追加文件

Exec source适用于监控一个实时追加的文件，不能实现断点续传；Spooldir Source适合用于同步新文件，但不适合对实时追加日志的文件进行监听并同步；而Taildir Source适合用于监听多个实时追加的文件，并且能够实现断点续传。

**1.案例需求：使用Flume监听整个目录的实时追加文件，并上传至HDFS**

**2.需求分析：**

![img](file:///C:/Users/whj/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

**3.实现步骤：**

（1）创建配置文件flume-taildir-hdfs.conf

创建一个文件

```
[nogc@hadoop102 job]$ vim flume-taildir-hdfs.conf
```

添加如下内容:

```
a3.sources = r3
a3.sinks = k3
a3.channels = c3

# Describe/configure the source
a3.sources.r3.type = TAILDIR
a3.sources.r3.positionFile = /opt/module/flume/tail_dir.json
a3.sources.r3.filegroups = f1 f2
a3.sources.r3.filegroups.f1 = /opt/module/flume/files/.*file.*
a3.sources.r3.filegroups.f2 = /opt/module/flume/files/.*log.*

# Describe the sink
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path = hdfs://hadoop102:9820/flume/upload2/%Y%m%d/%H
#上传文件的前缀
a3.sinks.k3.hdfs.filePrefix = upload-
#是否按照时间滚动文件夹
a3.sinks.k3.hdfs.round = true
#多少时间单位创建一个新的文件夹
a3.sinks.k3.hdfs.roundValue = 1
#重新定义时间单位
a3.sinks.k3.hdfs.roundUnit = hour
#是否使用本地时间戳
a3.sinks.k3.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a3.sinks.k3.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a3.sinks.k3.hdfs.fileType = DataStream
#多久生成一个新的文件
a3.sinks.k3.hdfs.rollInterval = 60
#设置每个文件的滚动大小大概是128M
a3.sinks.k3.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a3.sinks.k3.hdfs.rollCount = 0

# Use a channel which buffers events in memory
a3.channels.c3.type = memory
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.r3.channels = c3
a3.sinks.k3.channel = c3
```

![img](file:///C:/Users/whj/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

（2）启动监控文件夹命令

```
[nogc@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name a3 --conf-file job/flume-taildir-hdfs.conf
```

（3）向files文件夹中追加内容

在/opt/module/flume目录下创建files文件夹

```
[nogc@hadoop102 flume]$ mkdir files
```

向upload文件夹中添加文件

```
[nogc@hadoop102 files]$ echo hello >> file1.txt
[nogc@hadoop102 files]$ echo nogc >> file2.txt
```

（4）查看HDFS上的数据

![image-20200504220452482](C:\Users\whj\AppData\Roaming\Typora\typora-user-images\image-20200504220452482.png)

**Taildir说明：**

Taildir Source维护了一个json格式的position File，其会定期的往position File中更新每个文件读取到的最新的位置，因此能够实现断点续传。Position File的格式如下：

```
{"inode":2496272,"pos":12,"file":"/opt/module/flume/files/file1.txt"}
{"inode":2496275,"pos":12,"file":"/opt/module/flume/files/file2.txt"}
```

注：Linux中储存文件元数据的区域就叫做inode，每个inode都有一个号码，操作系统用inode号码来识别不同的文件，Unix/Linux系统内部不使用文件名，而使用inode号码来识别文件。