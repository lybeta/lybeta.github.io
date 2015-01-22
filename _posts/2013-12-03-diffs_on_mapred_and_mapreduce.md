---
layout: post
title: "Hadoop中mapred包和mapreduce包的区别"
category: 技术
tags: [Hadoop]
---


1.    首先第一条，也是我今天碰到这些问题的原因，新旧API不兼容。所以，以前用旧API写的hadoop程序，如果旧API不可用之后需要重写，也就是上面我的程序需要重写，如果旧API不能用的话，如果真不能用，这个有点儿小遗憾！

2.    新的API倾向于使用抽象类，而不是接口，使用抽象类更容易扩展。例如，我们可以向一个抽象类中添加一个方法(用默认的实现)而不用修改类之前的实现方法。因此，在新的API中，Mapper和Reducer是抽象类。

	<!--more-->

3.    新的API广泛使用context object(上下文对象)，并允许用户代码与MapReduce系统进行通信。例如，在新的API中，MapContext基本上充当着JobConf的OutputCollector和Reporter的角色。

4.    新的API同时支持"推"和"拉"式的迭代。在这两个新老API中，键/值记录对被推mapper中，但除此之外，新的API允许把记录从map()方法中拉出，这也适用于reducer。分批处理记录是应用"拉"式的一个例子。

5.    新的API统一了配置。旧的API有一个特殊的JobConf对象用于作业配置，这是一个对于Hadoop通常的Configuration对象的扩展。在新的API中，这种区别没有了，所以作业配置通过Configuration来完成。作业控制的执行由Job类来负责，而不是JobClient，并且JobConf和JobClient在新的API中已经荡然无存。这就是上面提到的，为什么只有在mapred中才有Jobconf的原因。

6.   输出文件的命名也略有不同，map的输出命名为part-m-nnnnn，而reduce的输出命名为part-r-nnnnn，这里nnnnn指的是从0开始的部分编号。

这样了解了二者的区别就可以通过程序的引用包来判别新旧API编写的程序了。个人建议最好用新的API编写hadoop程序，以防旧的API被抛弃！！！