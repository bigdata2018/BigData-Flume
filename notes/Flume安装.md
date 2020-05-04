# Flume安装部署

<nav>
    <a href="#一、前置环境">一、前置环境</a><br/>
<a href="#二、Flume安装地址">一、Flume安装地址</a><br/>
<a href="#三、安装部署">二、安装部署</a><br/>
</nav>



## 一、前置环境

**1.Hadoop3.1.3安装**



## 二、Flume安装地址

**1.Flume官网地址：http://flume.apache.org/**

**2.文档查看地址：http://flume.apache.org/FlumeUserGuide.html**

**3.下载地址：http://archive.apache.org/dist/flume/**





## 三、安装部署

**1.将apache-flume-1.9.0-bin.tar.gz上传到linux的/opt/software目录下**

**2.解压apache-flume-1.9.0-bin.tar.gz到/opt/module/目录下**

```shell
tar -zxf /opt/software/apache-flume-1.9.0-bin.tar.gz -C /opt/module/
```

**3.修改apache-flume-1.9.0-bin的名称为flume**

```shell
mv /opt/module/apache-flume-1.9.0-bin /opt/module/flume
```

4.将lib文件夹下的guava-11.0.2.jar删除以兼容Hadoop 3.1.3

```shell
rm /opt/module/flume/lib/guava-11.0.2.jar
```

