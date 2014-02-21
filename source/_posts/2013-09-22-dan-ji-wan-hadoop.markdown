---
layout: post
title: "单机玩Hadoop"
date: 2013-09-22 18:11
comments: true
categories: Server
---
以前在Amazon Web Service[AWS](http://aws.amazon.com/) 做过Hadoop 运算，处理业务逻辑，当时也曾在自己电脑做一个单一的节点模拟.在Tencent 有机会处理tencent 游戏的海量数据分析，这时候用到的是公司的TDW,也是基于Hadoop 的改造。大数据被炒的火热，特别是某些公司会把这些当作自我标榜更是让人恶心.本着折腾的信，把自己玩hadoop的过程写下来 ：）

+ 创建hadoop用户组;

```
sudo addgroup hadoop
sudo adduser -ingroup hadoop hadoop
```

给hadoop用户添加权限，编辑/etc/sudoers文件; 在root   ALL=(ALL:ALL)   ALL下

```
hadoop  ALL=(ALL:ALL) ALL
```

+ 在Ubuntu下安装JDK 

```
sudo apt-add-repository ppa:flexiondotorg/java
sudo apt-get update
sudo apt-get install sun-java6-jre sun-java6-jdk sun-java6-plugin
```
如果你在第二条命令遇到错误:

```
W: GPG 错误：http://ppa.launchpad.net precise Release: 由于没有公钥，无法验证下列签名： NO_PUBKEY 2EA8F35793D8809A
请执行
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 2EA8F35793D8809A  
```

编辑 sudo vi /etc/environment
在其中添加如下两行：

```
JAVA_HOME=/usr/lib/jvm/java-6-sun
CLASSPATH=.:/usr/lib/jvm/java-6-sun/lib
```

+ 安装ssh 服务

```
sudo apt-get install ssh openssh-server
```

首先要转换成hadoop用户，执行以下命令：

```
su - hadoop
```

采用 rsa 方式 生成key

```
ssh-keygen -t rsa -P ""
```

进入~/.ssh/目录下，将id_rsa.pub追加到authorized_keys授权文件中,使其无密码登录本机

```
cd ~/.ssh
cat id_rsa.pub >> authorized_keys
```

+ 下载[hadoop](http://www.apache.org/dyn/closer.cgi/hadoop/common/),并安装

```
sudo cp hadoop-0.20.203.0rc1.tar.gz /usr/local/
```

解压hadoop-0.20.203.tar.gz；

```
cd /usr/local
sudo tar -zxf hadoop-0.20.203.0rc1.tar.gz
```

将解压出的文件夹改名为hadoop;

```
sudo mv hadoop-0.20.203.0 hadoop
```

将该hadoop文件夹的属主用户设为hadoop，

```
sudo chown -R hadoop:hadoop hadoop
```

打开hadoop/conf/hadoop-env.sh文件;

```
sudo vi hadoop/conf/hadoop-env.sh
```

配置conf/hadoop-env.sh,配置本机jdk的路径;

```
export JAVA_HOME=/usr/lib/jvm/java-6-sun/
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:/usr/local/hadoop/bin
```

记得source hadoop-env.sh 

编辑conf/core-site.xml文件;

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!-- Put site-specific property overrides in this file. -->
<configuration>
    <property>  
        <name>fs.default.name</name>  
        <value>hdfs://localhost:9000</value>   
    </property>  
</configuration>
```

编辑conf/mapred-site.xml文件;

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!-- Put site-specific property overrides in this file. -->
<configuration>  
     <property>   
           <name>mapred.job.tracker</name>  
            <value>localhost:9001</value>   
      </property>  
</configuration>
```

编辑conf/hdfs-site.xml文件;

```
<configuration>
    <property>
        <name>dfs.name.dir</name>
        <value>/usr/local/hadoop/datalog1,/usr/local/hadoop/datalog2</value>
    </property>
    <property>
        <name>dfs.data.dir</name>
        <value>/usr/local/hadoop/data1,/usr/local/hadoop/data2</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
</configuration>
```

+ 运行hadoop,并启动

```
cd /usr/local/hadoop/
bin/hadoop namenode -format // 进入hadoop目录下，格式化hdfs文件系统
bin/start-all.sh //启动脚本
```

检测hadoop是否启动成功

```
hadoop@hp:/usr/local/hadoop/bin$ sudo jps
11822 DataNode
12461 Jps
11581 NameNode
12157 JobTracker
12064 SecondaryNameNode
12377 TaskTracker
```

说明hadoop单机版环境配置好了
此时访问[http://localhost:50030/](http://localhost:50030/),便可以看到管理界面
{% img /images/2013/09/hadoop.png %}
让我们来完成那篇著名论文的wordcounw吧

创建输入文件夹,并挂载hdfs

```
hadoop@hp:/usr/local/hadoop$ mkdir input //创建输入文件夹
hadoop@hp:/usr/local/hadoop$ cp conf/* input
hadoop@hp:/usr/local/hadoop$ bin/hadoop fs -put input/ input
```

执行wordcount程序,输入为input，输出数据目录为output。,

```
bin/hadoop jar hadoop-examples-0.20.203.0.jar wordcount input output
```

出现了运行情况如下

```
13/09/22 17:59:40 INFO input.FileInputFormat: Total input paths to process : 15
13/09/22 17:59:41 INFO mapred.JobClient: Running job: job_201309221738_0006
13/09/22 17:59:42 INFO mapred.JobClient:  map 0% reduce 0%
13/09/22 17:59:58 INFO mapred.JobClient:  map 13% reduce 0%
13/09/22 18:00:07 INFO mapred.JobClient:  map 26% reduce 0%
13/09/22 18:00:16 INFO mapred.JobClient:  map 40% reduce 8%
13/09/22 18:00:22 INFO mapred.JobClient:  map 53% reduce 13%
13/09/22 18:00:28 INFO mapred.JobClient:  map 66% reduce 13%
13/09/22 18:00:31 INFO mapred.JobClient:  map 66% reduce 17%
13/09/22 18:00:34 INFO mapred.JobClient:  map 80% reduce 17%
13/09/22 18:00:37 INFO mapred.JobClient:  map 80% reduce 22%
13/09/22 18:00:40 INFO mapred.JobClient:  map 93% reduce 22%
13/09/22 18:00:46 INFO mapred.JobClient:  map 100% reduce 26%
13/09/22 18:00:55 INFO mapred.JobClient:  map 100% reduce 100%
13/09/22 18:01:00 INFO mapred.JobClient: Job complete: job_201309221738_0006
13/09/22 18:01:00 INFO mapred.JobClient: Counters: 25
13/09/22 18:01:00 INFO mapred.JobClient:   Job Counters 
```

查看执行结果

```
hadoop@hp:/usr/local/hadoop$ bin/hadoop fs -cat output/*
```

截取部分结果现场

```
hadoop@hp:/usr/local/hadoop$ hadoop fs -cat output/*
"". 4
"*" 10
"alice,bob  10
"console"   1
"hadoop.root.logger".   1
"jks".  4
   79
$HADOOP_BALANCER_OPTS"   1
$HADOOP_DATANODE_OPTS"   1
$HADOOP_HOME/conf/slaves 1
$HADOOP_HOME/logs    1
$HADOOP_JOBTRACKER_OPTS" 1
$HADOOP_NAMENODE_OPTS"   1
$HADOOP_SECONDARYNAMENODE_OPTS"  1
```

停止hadoop

```
hadoop@hp:/usr/local/hadoop$ ./stop-all.sh 
```

hadoop这只小象在单机可以这么玩


