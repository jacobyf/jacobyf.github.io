---
layout: post
title: "Hadoop 1.2.1 集群配置"
categories: coding
date: 2014-02-28 20:02:00

excerpt: "Hadoop 1.2.1安装配置笔记，你懂的。"

author:
  name: Jacob Yang
---

# Hadoop1.2.1 安装配置笔记

## 预置环境条件
1.Java环境准备，最好使用jdk1.6.x，Sun的JDK最好，将下好的`jdk-6u45-linux-x64.bin`解压

    $ sudo ./jdk-6u45-linux-x64.bin
    $ sudo mkdir /usr/local/java
    $ sudo cp jdk1.6.0_45 /usr/local/java
    $ sudo ln -s jdk1.6.0_45 jdk
  
2.SSH环境，通过SSH，可以通过Hadoop脚本操作远程的daemons
## 配置统一的Hadoop帐号

1.建立统一的Hadoop帐号，密码'hezuo'
  
    $ sudo addgroup hadoop
    $ sudo adduser --ingroup hadoop hadoop
2.Hadoop主机名'hadoop-master'
  
    $ sudo vim /etc/hostname

## 配置Hadoop
1.解压hadoop1.2.1的压缩包
    
    $ sudo tar -xzvf hadoop-1.2.1.tar.gz
    $ sudo cp hadoop-1.2.1 /usr/local
    $ cd /usr/local
    $ sudo ln -s hadoop-1.2.1 hadoop
2.将hadoop的所有者赋予hadoop用户和hadoop用户组

    $ sudo chown -Rv hadoop:hadoop hadoop-1.2.1
      
## 配置SSH
1.以Hadoop帐号登录后，使用如下命令生成RSA密钥对。生成`~/.ssh/id_rsa`和`~/.ssh/id_rsa.pub`两人个文件。并输入空密码。
    
    $ ssh-keygen -t rsa -f ~/.ssh/id_rsa
      
2.将产生的公钥共享出去

    $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
      
## Hadoop配置文件
### 配置文件列表，这些文件都在hadoop安装目录下的`conf`目录中。
* hadoop-env.sh (Hadoop脚本运行环境配置)
* core-site.xml (HDFS\MapReduce常用配置)
* hdfs-site.xml (Hadoop守护进程配置项，包括NameNode, SNN, DataNode配置)
* mapred-site.xml (MapReduce守护进程配置项，包括jobTracker和taskTracker等)
* masters (运行Secondary NameNode列表配置, 这个文件主要由主机维护) 
* slaves (运行DataNode和TaskTracker的机器列表，这个文件主要由主机维护)

### 配置hadoop-env.sh，在里面配置一个`JAVA_HOME`环境变量
    $ vim /usr/local/hadoop/conf/hadoop-env.sh
    input: export JAVA_HOME=/usr/local/java/jdk

### 配置core-site.xml文件
    <configuration>
      <property>
        <name>fs.default.name</name>
        <value>hdfs://hadoop-master:9000</value>
      </property>
    </configuration>
### 配置hdfs-site.xml  
先创建HDFS目录  

    $ cd ~
    $ mkdir -p hadoop-hdfs/name
    $ mkdir -p hadoop-hdfs/data
    
然后配置hdfs-site.xml文件

    <property>
     	<name>dfs.name.dir</name>
     	<value>file:/home/hadoop/hadoop-hdfs/name</value>
    </property>
    <property>
     	<name>dfs.data.dir</name>
     	<value>file:/home/hadoop/hadoop-hdfs/data</value>
    </property>

### 配置mapred-site.xml文件
    $ mkdir -p hadoop-hdfs/mapred/local
    $ mkdir -p hadoop-hdfs/mapred/system
    <property>
      <name>mapred.job.tracker</name>
      <value>hadoop-master:8021</value>
      <final>true</final>
    </property>
    <property>
      <name>mapred.local.dir</name>
      <value>file:/home/hadoop/hadoop-hdfs/mapred/local</value>
      <final>true</final>
    </property>
    <property>
      <name>mapred.system.dir</name>
      <value>file:/home/hadoop/hadoop-hdfs/mapred/system</value>
      <final>true</final>
    </property>
    <property>
      <name>mapred.tasktracker.map.tasks.maximum</name>
      <value>7</value>
      <final>true</final>
    </property>
     <property>
      <name>mapred.tasktracker.reduce.tasks.maximum</name>
      <value>7</value>
      <final>true</final>
    </property>
    
### 配置masters文件，配置一个SNN(Secondary Name Node)节点的主机列表

    $ vim /usr/local/hadoop/conf/masters
    input: hadoop-master

### 配置slaves文件，配置数据节点和taskTrackers节点的主机列表

    $ vim /usr/local/hadoop/conf/slaves
    input: hadoop-slave
    ...

## 运行Hadoop

    $ start-dfs.sh # 启动HDFS
    $ start-mapred.sh # 启动MapReduce
    
## 判断运行效果:使用jps查看运行的daemon进程

    $ jps
    
      
