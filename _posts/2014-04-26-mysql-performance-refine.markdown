---
layout: post
title: "MySQL怎么用才能锋利——性能优化总结"
date: 2014-04-26 11:42:00
categories: [database, mysql]
tags: [SQL, MySQL]

excerpt: "前一段时间在使用客户的数据进行数据挖掘分析的时候，发现使用的数据规模不再是以前接触的数十万、数百万的级别，所以在处理数据的时候吃了不少苦头"

author:
  name: Jacob Yang
---


> 前一段时间在使用客户的数据进行数据挖掘分析的时候，发现使用的数据规模不再是以前接触的数十万、数百万的级别，所以在处理数据的时候吃了不少苦头。 可能有同学会问，为何不用Oracle？主要还是计算资源的问题，
虽然使用的SUSE Linux所处的环境是128G、16核的单板，但Oracle装上之后会占用几乎所有的资源。实话说，一时半会也没有其它的计算资源，所以跟同伴决定用MySQL，但MySQL又是相当tricky的事情。不可避免地，我们与
MySQL死磕上了。

## 既定的使用场景
不同于一般的Client与Server的应用请求场景，我所需要训练出的是一个高性能、高效率的数据挖掘工具，并行连入数据库的客户端在一个3-5人的小团队内。我们的目标不需要考虑太多用户连入的问题，而是考虑怎么让MySQL
用上足够多的硬件资源，达到最大的利用率。因此，一些优化策略可能并不适用于大型的线上应用（LAMP）。

### MyISAM向左，Innodb向右
MySQL支持多个引擎，常用的有`MyISAM`，`Innodb`，`CSV`，`ARCHIVE`等。经常又能拿出来比较的呢，主要就是`MyISAM`与`Innodb`了。那到底是选择`MyISAM`还是`Innodb`呢？主要还是看你怎么回答以下几个问题。

* 是否需要一个支持原子事务的数据库？
* 是否需要一个查询效率最快的数据库？
* 你是否会经常做数据更新、删除操作？

可以肯定的是，`MyISAM`在查询性能上是比`Innodb`好的，但它又不支持事务处理，而且官方文档中也说到最好少做数据的更新、删除。可能对于一般小型的电子商务系统而言，不支持事务是不可能的事情，但我们的任务不一样。我们不要求
事务的原子性，我们目标就是效率、最大效能。所以，毫无疑问，`MyISAM`是最佳选择，而且它在非并行的数据插入也是比`Innodb`要好！

有关`MyISAM`与`Innodb`的比较可以查看下面链接：

* [MyISAM数据库引擎][0-1]
* [Innodb数据库引擎][0-2]
* [MySQL:Innodb还是MyISAM?][0-3]
* [MySQL Engines - MyISAM vs Innodb][0-4]

### 数据导入场景
由于使用的数据都是客户的Oracle数据库里面的，在老的SUSE环境下也没有合适的Oracle导MySQL的工具，所以我们先是用Python脚本连入Oracle数据库，选择需要的数据导出成CSV格式的文件。一个月的话单数据，通过过滤，
总共导出了30个CSV文件，整体大小19G左右。接下来，怎么导入这些数据呢？

* 能够使用`LOAD DATA INFILE`导入数据，就不要使用`INSERT`语句，据MySQL官方文档了解，正常情况下`LOAD DATA INFILE`会比`INSERT`语法快几十倍。
* 如果要使用`INSERT`语法，不要将每一行的数据分别使用一条`INSERT`语句，而应该将几十条或几百条连到一个`INSERT`语句里执行。
* 使用第三方工具，推荐一个工具，网上都说不错[`mydumper`](https://launchpad.net/mydumper)。但由于我们在老SUSE下，安装起来很麻烦，我们最后放弃了。

### 优化字段索引创建速度
在做数据查询之前，一定要避免的就是全表扫描，特别是上亿级别的数据量下，全表扫描是很恐怖的一件事情。如果查询的字段中还包含部分聚合函数，MySQL还会创建临时表，而MySQL默认在内存里创建临时表的大小仅有16M，那么一个10G左右的表会重新在磁盘上生成一个相同大小的临时表进行重排，这个速度你肯定受不了。

所以，索引的创建是必须的。同样，由于表的基础数据量如此庞大的情况下，创建索引也不是一件很容易的事情。怎么加快MySQL创建MYISAM表的索引呢？下面有几个重要配置需要加上。

* `myisam_max_sort_file_size`，这个选项可以配置MYISAM表允许在创建索引的过程中，能够往磁盘创建一个临时索引文件的大小，这个可以配置大一点，比如一个10G大小的表，可能生成的索引文件有4~5G。
* `myisam_repair_threads`，这个选项可以在创建索引的时候同时开启多个线程，不过前提是，每一个线程只负责创建一个索引。
* `read_buffer_size`，在MYISAM表创建索引的时候，MySQL会先将原来的表拷贝一个临时表，然后在这个临时表的基础上创建，这个选项可以控制这个拷贝速度。
* `key_buffer_size`，在创建索引的时候，也可以将这个选项调大一点，这个能够让更多索引创建过程在内存里进行，而不需要频繁访问磁盘。

以上几个选项，都可以在`my.ini`（Linux下是在`my.cnf`）里做配置，当然这里会对全局有影响，如果你只是让作用只局限在创建索引的过程中，你可以在SQL语句中将这些变量设置加入SESSION作用域中。
    
    SET SESSION myisam_max_sort_file_size = 5 * 1024 * 1024  * 1024; # 5G
    SET SESSION myisam_repair_threads = 1;
    SET SESSION read_buffer_size = 256 * 1024 * 1024; # 256M
    SET SESSION key_buffer_size = 128 * 1024 * 1024; # 128M
    ALTER TABLE table_name ADD INDEX index_name (col_name, ... col_name);
        
### 数据查询优化策略
在做数据查询的时候，建议按部就班地按下面几个步骤进行，盲目地去执行一条看似优美的SQL语句不可取，代码之美首先要可用性、可靠性。

建立索引，面对千万上亿级的数据量，建立索引应该是条件反射的。索引是创建单独的索引还是联合索引？这个主要是基于你经常要用到的查询去考虑。假设一个表有三个字段`a`, `b`, `c`,你经常需要基于这个三个字段进行聚合，经常需要用到`count`, `max`等聚合函数，最好的策略是在这三个字段上建立联合索引

使用explain语法查询执行计划。与其病入膏肓，烂到肠子才知道痛，还不如一开始就了解病症的发端。数据量一大，很可能一条SQL语句执行个半个小时都没有反应，那最后能知道这条SQL语句是怎么执行的。MySQL就提供了一个`explain`语法用于查询SQL执行计划。那使用`explain`又应该关注什么呢？没错，还是关注你的SQL语句是使用索引了呢？还是要创建临时表？或是需要用到`fileSort`。


通过SQL执行计划，我们了解到，表查询一旦用不上索引，就会需要使用创建临时表，在临时表上进行数据重排。既然临时表不可避免，那就尽全力让临时表的操作更多使用内存资源去做，只要你的内存够用。那怎么做？我们不妨查看一下MySQL是如何管理内存表的创建的。

从上面可以了解到，MySQL具有两个配置变量`max_heap_table_size`与`tmp_table_size`，这两个变量的最小值决定能够在内存中创建多大的临时表，默认值是16M。16M可能对于几万的数据差不多，但当数据上个千万级怎么够用？所以，按使用的一表的大小来看，我将这两个值都改到了5G的大小，效果也很明显，原来半个小时都出不来结果，修改这两个变量后大概6分钟左右就能够将查询结果得出。

这两个变量怎么设置呢？还是在两个地方设置，一个是将它们设置在`my.ini`里，这样可以全局可用或者是在查询中设置一个`SESSION`变量。最后，还
注意，这个值也不设置地越大越好，比如你一个300M大小的表，你非要设置个10G的临时表创建阈值，也太浪费，另外在内存上还得耗费寻址时间。

有关查询优化这一块，请另外参考一下MySQL官方指引:

* [使用Explain查询执行计划][1-1]
* [优化索引使用][1-2]
* [优化查询缓存策略][1-3]

### 一些有用的工具语法
下面再介绍一些经常能够用到的一些工具式语法。

* 查询当前MySQL当前运行的SQL任务，使用`mysqladmin`查询`processlist`或直接使用`show processlist`。

        mysqladmin -uroot -p123 processlist; # 这种方式会展示出当前的SQL任务列表
        
        while :;do mysqladmin -uroot -p123 processlist 2>/dev/null && sleep 5 && clear; done; # 这样可以每隔5秒刷新一次任务状态列表 
* 在做数据插入的过程中，查询当前表插入的行数（对于Innodb可以实时获取结果，MyISAM插入过程中如果锁定全表，则只能等最后获取到结果）。

        mysql -uroot -p123 -e "select table_rows from information_schema.tables where table_name = 'TABLE_NAME'";
* 查询MySQL配置变量默认值，（前面也用到过的`show variables like`）
        
        show variables like '%buffer_size%';
* 查询MySQL的执行计划

        explain SQL_STATEMENT;
* 查询表包括的索引信息

        show index from TABLE;
        

[0-1]: https://dev.mysql.com/doc/refman/5.0/en/myisam-storage-engine.html
[0-2]: https://dev.mysql.com/doc/refman/5.0/en/innodb-storage-engine.html
[0-3]: http://coolshell.cn/articles/652.html
[0-4]: http://www.rackspace.com/knowledge_center/article/mysql-engines-myisam-vs-innodb
[1-1]: https://dev.mysql.com/doc/refman/5.0/en/using-explain.html
[1-2]: https://dev.mysql.com/doc/refman/5.0/en/optimization-indexes.html
[1-3]: https://dev.mysql.com/doc/refman/5.0/en/buffering-caching.html