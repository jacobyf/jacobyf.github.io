---
layout: post
title: "从不同版本的Hadoop从迁移数据"
date: 2014-08-28 15:22:00
categories: [hadoop, bigdata]

excerpt: "如何从不同版本的Hadoop中迁移数据呢？可能许多朋友都知道使用`hadoop distcp`命令去从集群A拷贝到集群B。"
author:
  name: Jacob Yang
---

如何从不同版本的Hadoop中迁移数据呢？可能许多朋友都知道使用`hadoop distcp`命令去从集群A拷贝到集群B。
但有一点可能容易被忽略——集群A与集群B的Hadoop版本是否匹配呢？实验发现，不同版本的两个集群使用这
个命令远程拷贝数据成功率很低，许多数据在拷贝的过程中，你会发现好像全部拷贝完了，但马上文件大小又
变成了0，很是郁闷。为何？

原来，HDFS数据从不同集群拷贝完了之后，还需要做校验和，一但检验和不相等，所有前面传输的数据会被废弃
掉。Hadoop就使用了一个CRC校验机制。那原因都知道了，怎么解决这个问题呢？网上有其它比如修改配置文件，
打补丁的方式，我这里则使用了相对暴力的方式。
{% highlight shell %}
hadoop distcp -skipcrccheck -update hftp://clusterA:50070/path hdfs://clusterB:9000/path
{% endhighlight %}

当然，使用这种暴力模式是有前提的。由于我使用的两个集群环境是在同一个网段下，都是1000M网络环境，所以
基本上数据传输不匹配的风险比较小。另外，大家需要注意的是，使用`-skipcrccheck`一定要配套使用`-update`。
