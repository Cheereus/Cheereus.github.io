---
layout:     post
title:      Eclipse配置hadoop开发环境
subtitle:   Hadoop 踩坑记(三)
date:       2020-05-21
author:     Cheereus
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - 原创
    - Hadoop
    - Linux
    - CentOs
    - Eclipse
---

### 环境

windows 10

java 1.8

namenode(hadoop1-ali) 阿里云(CentOS 7.3) 120.26.173.104

hadoop 版本 `2.8.5`

### Eclipse 安装

需要安装 Enterprise 版本，由于网络原因，建议使用离线安装包

<https://www.eclipse.org/downloads/packages/release/2020-03/r/eclipse-ide-enterprise-java-developers-includes-incubating-components>

当然在此之前先要安装 java 在此不做赘述

### 配置 hadoop 插件

把下载好的 Hadoop 解压到本地目录。添加系统环境变量：新建变量名 `HADOOP_HOME`，值为Hadoop 的解压路径，如 `D:\Program Files\hadoop-2.7.5`

添加到 `path` 中：`%HADOOP_HOME%\bin`

把 `hadoop-eclipse-plugin-2.8.5.jar` 包复制到 Eclipse 目录下的 `plugins` 目录中。重启Eclipse。

打开 `Window->Prefences` 可以看到左侧多出了 `Hadoop Map/Reduce` 项

![插件配置1](/img/post/2020052201.jpg)

点击多出的 `Hadoop Map/Reduce` 项，在右侧添加 Hadoop 解压路径 如 `D:\Program Files\hadoop-2.7.5`

解压 `hadoop-common-2.8.5-bin-master` 包(本人是直接从 github 上找的，要寻找对应版本的文件)，把解压得到的 `bin` 目录下的 `hadoop.dll、hadoop.exp、hadoop.lib、winutils.exe` 这四个文件复制到 `hadoop-2.8.5` 的 `bin` 目录下。

再把 `hadoop.dll` 和 `winutils.exe` 复制到 `C:\Windows\System32` 目录下

### 配置hadoop连接

Eclipse 中依次点击：`Window->Open Perspective->Map/Reduce`，项目左侧中出现 `DFS Locations` 结构。

如果没有，直接新建一个 `map/reduce` 项目即可

![连接配置1](/img/post/2020052202.jpg)

Eclipse 中依次点击：`Window->Show View ->Other->MapReduce Tools->Map/Reduce Locations` 点击确定(open)

![连接配置2](/img/post/2020052203.jpg)

控制台多出了 `Map/Reduce Locations` 视图。

右键 `Map/Reduce Locations` 视图的空白处，选择新建，定义 Hadoop 集群的链接。

![连接配置3](/img/post/2020052204.jpg)

其中 location name 和 user name 随意

`Map/Reduce(V2) Master` 配置要与 hadoop 配置中 `mapred-site.xml` 内容保持一致

`DFS Master` 配置要与 `core-site.xml` 内容保持一致

点击 Finish 后 `DFS Locations` 下就会出现相关的连接信息

在配置正确的情况下，点开就能看到 hdfs 文件系统中的文件内容，即连接成功

![连接配置4](/img/post/2020052205.jpg)

### 运行wordcount示例

项目 `src` 下创建 Package(本文名为 wit)，Package 下创建 `WordCount.java` 类

粘贴如下 java 代码

```java
package wit;
import java.io.IOException;
import java.util.StringTokenizer;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser; 
public class WordCount {
    public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();
        public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                word.set(itr.nextToken());
                context.write(word, one);
            }
        }
    }
    public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
        private IntWritable result = new IntWritable();
        public void reduce(Text key, Iterable<IntWritable> values, Context context)
                throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
        if (otherArgs.length != 2) {
            System.err.println("Usage: wordcount <in> <out>");
            System.exit(2);
        }
        @SuppressWarnings("deprecation")
        Job job = new Job(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

右键项目，依次 `Run as ->Run Configurations...->Java Application`

选 `Java Application` 后点击左上角的 `New launch application`，配置 Main 标签参数。

填写Name（任起），Search...往下拉，找到 WordCount，确定。

![示例配置1](/img/post/2020052206.png)

配置 Arguements

![示例配置2](/img/post/2020052207.png)

(此部分图来自<https://blog.csdn.net/u010185220/article/details/79095179>)

右键 WordCount 类，选择 `Build Path -> configure build path` 添加 jar 包

![示例配置3](/img/post/2020052208.jpg)

点击 `Add External JARs` 将 hadoop 解压文件夹中相关的 jar 包全部添加进来

配置完成，即可点击 Run 来运行 WordCount 类

运行结果会生成在前面配置的 hdfs 文件系统的指定目录中，可以直接在 Eclipse 中查看
