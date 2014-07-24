---
layout: post
title: "MySQL查询的时候选择rowid列"
date: 2014-04-03 09:44:00
categories: [database, mysql]
tags: [SQL, MySQL]
---

最近在编码的时候需要将一个数据集的列ID查询出来，对于这样的需求，不同数据库都给出了不同的方案。这里主要以MySQL为例，分享一下在MySQL下面怎么实现。

MySQL是没有相关的函数能做到这一点的，但是我们可以使用MySQL自定义变量。对！比如下面定义一个变量`@V_A`，然后使变量`@V_A`自增1，最终变量的值为31。
    
    SET @V_A = 30
    @V_A := @V_A + 1

好吧，变量准备好了，我们可以准备在查询中使用变量达到我们的目的。其实，我们只需要定义一个变量`@rank`，另外为了使`rowid`是从1开始递增，我们初始变量`@rank`的值为-1，然后在`SELECT`语句中使用到它。

    {% hightlight shell %}
    SET @rank = -1;
    
    SELECT @rank:=@rank+1 as row_id, name, age FROM table1 limit 5;
    
    ++++++++++++++++++++++++++
    + row_id + name   +  age +
    ++++++++++++++++++++++++++
    + 1      + Annie  +  23  +
    ++++++++++++++++++++++++++
    + 2      + Jacob  +  24  +
    ++++++++++++++++++++++++++
    + 3      + Jim    +  22  +
    ++++++++++++++++++++++++++
    + 4      + Rank   +  25  +
    ++++++++++++++++++++++++++
    + 5      + Red    +  20  +
    ++++++++++++++++++++++++++
    {% endhighlight %}
    
这是一种查询方式，另外还有一个变体，我们可以充分使用`JOIN`进行表连接查询实现。

    {% hightlight shell %}
    SELECT @rank:=@rank+1 as row_id, name, age FROM table1 INNER JOIN (SELECT @rank:=-1) v;
    {% endhighlight %}
    
    
如何，这个查询感觉比上一个更紧凑一点吧。

    {% highlight ruby %}
    def show
      @widget = Widget(params[:id])
      respond_to do |format|
        format.html # show.html.erb
        format.json { render json: @widget }
      end
    end
    {% endhighlight %}
    
