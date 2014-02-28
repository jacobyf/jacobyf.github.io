---
layout:post
title: "Hadoop单节点配置"
---
# Hadoop配置笔记

## Hadoop单机配置
### 预置环境
主要准备jdk与ssh环境
    $ sudo apt-get install openjdk-7-jdk
    $ java -version
    $ cd /usr/lib/jvm/
    $ sudo ln -s java-7-openjdk-amd64 jdk
    $ sudo apt-get install openssh-server
  
### 添加Hadoop用户
    $ sudo addgroup hadoop
    $ sudo adduser --ingroup hadoop hduser
    $ sudo adduser hduser sudo
以上这里添加用户组的通用操作，实际这里的添加操作如下。
    $ sudo addgroup hadoop
    $ sudo adduser hezuo hadoop
  
### 设置SSH证书(Certificate)
    $ ssh-keygen -t rsa
    $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    $ ssh localhost ?
### 下载并安装Hadoop
    $ cd ~
    $ sudo tar vxzf hadoop-2.2.0.tar.gz -C /usr/local
    $ cd /usr/local
    $ sudo ln -s hadoop-2.2.0 hadoop
    $ sudo chown -R hezuo:hadoop hadoop

### 环境变量配置
    $ cd ~
    $ vim .bashrc
    
设置以下环境变量  

    export JAVA_HOME=/usr/lib/jvm/jdk/
    export HADOOP_INSTALL=/usr/local/hadoop
    export PATH=$PATH:$HADOOP_INSTALL/bin
    export PATH=$PATH:$HADOOP_INSTALL/sbin
    export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
    export HADOOP_COMMON_HOME=$HADOOP_INSTALL
    export HADOOP_HDFS_HOME=$HADOOP_INSTALL
    export YARN_HOME=$HADOOP_INSTALL
然后再到Hadoop目录下进行设置   

    $ cd /usr/local/hadoop/etc/hadoop
    $ vim hadoop-env.sh
修改JAVA_HOME  

    export JAVA_HOME=/usr/lib/jvm/jdk
  
### 配置Hadoop
* 配置core-site.xml,打开/usr/local/hadoop/etc/hadoop/core-site.xml，在`<configuration>`节点中添加如下代码:  

        <property>
          <name>fs.default.name</name>
          <value>hdfs://localhost:9000</value>
        </property>
* 配置yarn-site.xml,打开/usr/local/hadoop/etc/hadoop/yarn-site.xml，在`<configuration>`节点中添加如下代码：  

        <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
        </property>
        <property>
          <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
          <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
* 配置mapred-site.xml，先拷贝mapred-site.xml.template为mapred-site.xml，然后通过gedit打开。　

        $ cd /usr/local/hadoop/etc/hadoop
        $ sudo cp mapred-site.xml.template mapred-site.xml
        $ sudo gedit mapred-site.xml
接着，拷贝如下代码到`<configuration>`节点中。　

      <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
      </property>    
    
* 用如下命令创建“名字节点”和“数据节点”　

        $ cd ~
        $ mkdir -p hadoop/hdfs/namenode
        $ mkdir -p hadoop/hdfs/datanode
* 使用gedit编辑文件`/usr/local/hadoop/etc/hadoop/hdfs-site.xml`　

        $ sudo gedit /usr/local/hadoop/etc/hadoop/hdfs-site.xml
        
        <property>
          <name>dfs.replication</name>
        	<value>1</value>
        </property>
        <property>
         	<name>dfs.namenode.name.dir</name>
         	<value>file:/home/hezuo/hadoop/hdfs/namenode</value>
        </property>
        <property>
         	<name>dfs.datanode.data.dir</name>
         	<value>file:/home/hezuo/hadoop/hdfs/datanode</value>
        </property>

* 使用HDFS格式化“名字节点(namenode)”　

          $ hdfs namenode -format
    
### 启动Hadoop服务
* 使用如下SHELL命令启动　　

        $ start-dfs.sh
        ...
        $ start-yarn.sh
        ...
* 使用`jps`确认Hadoop服务顺利运行　

        $jps
如果一切顺利，会显示　

        2583 DataNode
        2970 ResourceManager
        3461 Jps
        3177 NodeManager
        2361 NameNode
        2840 SecondaryNameNode
