---
layout:     post
title:      在Centos7上部署hadoop集群
subtitle:   Hadoop 踩坑记(一)
date:       2020-05-18
author:     Cheereus
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - 原创
    - Hadoop
    - Linux
    - CentOs
---

### 环境

机器1(hadoop1-ali) 阿里云(CentOS 7.3) 120.26.173.104  

机器2(hadoop2-hw) 华为云(CentOS 7.4) 114.116.233.156

其中第一台服务器用作 namenode 第二台用作 datanode

### 修改 hostname 及 host 文件

分别在两台机器上执行

```shell
hostname hadoop1-ali
```

```shell
hostname hadoop2-hw
```

修改两台机器的 `/etc/hosts` (host 要带 s)文件，添加以下内容

```shell
120.26.173.104 hadoop1-ali
114.116.233.156 hadoop2-hw
```

修改之后可以分别检查 host 是否生效

```shell
ping hadoop1-ali
ping hadoop2-hw
```

### 给两台机器生成秘钥文件

分别在两台机器上运行命令生成 ssh 秘钥

```shell
ssh-keygen -t rsa -P ''
```

新建一个名为 `authorized_keys` 的文件

将每台机器中 `/root/.ssh/id_rsa.pub` 文件的内容都复制到上述文件中，每台一行

然后将 `authorized_keys` 文件上传到每台机器的 `/root/.ssh/` 目录下

成功后测试使用ssh进行无密码登录，例如在 hadoop1-ali 上测试登录另一台机器

```shell
ssh hadoop2-hw
```

输入一个 yes 之后就能看到另一台服务器的欢迎内容，至此两台机器的互相登录实现完毕

需要注意的是，上述操作都是用 root 用户进行的，这样省去权限配置等繁琐操作，但会带来潜在的安全问题，不建议在生产环境下直接使用 root 用户进行配置

### 安装 openJDK1.8

华为云默认已经安装了 openJDK1.8，因此只需在阿里云服务器上进行

```shell
yum install java-1.8.0-openjdk  -y
yum install java-1.8.0-openjdk-devel  -y
```

然后配置环境变量，编辑文件 `/etc/profile` 在最后加上

```shell
#Java
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64
export CALSSPATH=$JAVA_HOME/lib/*.*
export PATH=$PATH:$JAVA_HOME/bin
```

保存后使之生效

```shell
source /etc/profile
```

### 安装 hadoop

从 `https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.8.5/hadoop-2.8.5.tar.gz` 下载压缩包，上传到服务器的 `/opt/hadoop/` 目录下

解压之

```shell
tar -xvf hadoop-2.8.5.tar.gz
```

在 `/root` 目录下新建几个目录

```shell
mkdir  /root/hadoop
mkdir  /root/hadoop/tmp
mkdir  /root/hadoop/var
mkdir  /root/hadoop/dfs
mkdir  /root/hadoop/dfs/name
mkdir  /root/hadoop/dfs/data
```

### 修改 etc/hadoop 中的一系列配置文件

进入目录 `/opt/hadoop/hadoop-2.8.5/etc/hadoop`

修改 `core-site.xml` 在 `<configuration>` 节点内加入配置

```shell
<configuration>
   <property>

        <name>hadoop.tmp.dir</name>

        <value>/root/hadoop/tmp</value>

        <description>Abase for other temporary directories.</description>

   </property>

   <property>

        <name>fs.default.name</name>

        <value>hdfs://hadoop1-ali:9000</value>

   </property>
</configuration>
```

修改 `hadoop-env.sh` 将 `${JAVA_HOME}` 修改为自己的 jdk 路径

```shell
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64
```

修改 `hdfs-site.xml` 加入配置

```shell
<property>

   <name>dfs.name.dir</name>

   <value>/root/hadoop/dfs/name</value>

   <description>Path on the local filesystem where theNameNode stores the namespace and transactions logs persistently.</description>

</property>

<property>

   <name>dfs.data.dir</name>

   <value>/root/hadoop/dfs/data</value>

   <description>Comma separated list of paths on the localfilesystem of a DataNode where it should store its blocks.</description>

</property>

<property>

   <name>dfs.replication</name>

   <value>2</value>

</property>

<property>

      <name>dfs.permissions</name>

      <value>false</value>

      <description>need not permissions</description>

</property>
```

新建并且修改 `mapred-site.xml`

复制目录中的 template 文件并重命名

```shell
cp mapred-site.xml.template mapred-site.xml
```

加入配置

```shell
<property>

   <name>mapred.job.tracker</name>

   <value>hadoop1-ali:49001</value>

</property>

<property>

      <name>mapred.local.dir</name>

       <value>/root/hadoop/var</value>

</property>


<property>

       <name>mapreduce.framework.name</name>

       <value>yarn</value>

</property>
```

修改 `slaves` 文件，将其中的 localhost 删除并添加自己的 datanode

```shell
hadoop2-hw
```

修改 `yarn-site.xml` 文件，加入配置

```shell
<property>

        <name>yarn.resourcemanager.hostname</name>

        <value>hadoop1-ali</value>

   </property>

   <property>

        <name>yarn.nodemanager.aux-services</name>

        <value>mapreduce_shuffle</value>

   </property>

```

在另一台机器上重复上述操作，文件可以直接复制，涉及六个文件，两台服务器上保持完全一致

`core-site.xml, mapred-site.xml, yarn-site.xml, slaves, hadoop-env.sh，hdfs-site.xml`

其中涉及到 jdk 路径需要修改为 `java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.aarch64`

### 启动 hadoop

在 namenode(hadoop1-ali) 上执行初始化

```shell
cd /opt/hadoop/hadoop-2.8.5/bin
./hadoop namenode -format
```

在 namenode(hadoop1-ali) 上执行启动命令

```shell
cd /opt/hadoop/hadoop-2.8.5/sbin
./start-all.sh
```

输入两次 yes 后即可成功

分别在阿里云和华为云后台修改安全组开放端口

在浏览器访问 `http://120.26.173.104:50070/` 中即可查看效果

### 补充说明

文中涉及的 hostname、ip、jdk 等内容因人和设备不同而有所不同

后续经过测试，`hosts` 文件中的节点自身 ip 应该填内网 ip

### 运行 wordcount 示例

先要配置 hadoop 环境变量，此部分直接在后面的参考内容

在hdfs文件系统中建立input目录

```shell
hdfs dfs -mkdir /input
```

新建一个 `example` 文件并写入任意语句

将本地的 `example` 文件复制到 hdfs 文件系统的 `/input` 目录中

```shell
hdfs dfs -copyFromLocal example /input
```

在 `hadoop-2.8.5` 的目录中运行

```shell
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.5.jar wordcount /input /output
```

WordCount 为 mapreduce 的一个示例程序，用于统计 `/input` 目录中所有文件的单词个数，并将结果存储在 `/output` 目录中。

运行结束后，查看单词统计结果

```shell
hdfs dfs -cat /output/*
```

### 参考

流程主要参考

<https://blog.csdn.net/pucao_cug/article/details/71698903>

`hosts` 文件中的本机 ip 应该填内网 ip，否则会引发 namenode 启动失败，此问题参考自

<https://blog.csdn.net/dongdong9223/article/details/81255713>

hadoop 环境变量配置

<https://blog.csdn.net/fantasydreams/article/details/4648649>
