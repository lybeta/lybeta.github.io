---
layout: post
title: "关于子查询和连接的点滴记录"
category: 技术
tags: [Sql]
---



刚看到一个关于sql执行效率的比较，如下

	A：
		select title
		from simplified
		where id in (select id
		                  from analysis
		                  where word = ‘something’);
	B：
		select b.title
		from  analysis a
		join simplified b
		on (a.id=b.id)
		where a.word=’something’;
	C：
		select simplified.title
		from  analysis
		join simplified
		on (analysis.id=simplified.id)
		where analysi.word=’something’;

由文章总结出一下几点：

<!--more-->

* 子查询适合外结果集大，子查询结果集小的情况，最好是能保证子查询所返回的结果集尽量的小。
* 在数据量差不多，都是万条记录左右的情况下，A应该会慢一点。 如果是在mysql上执行的话，A中子查询语句会认为与外面的simolified表进行关联比较。这样的话A其实就回被翻译成：

		select title from simplified where exists (select simplified.title from analysis where word= ‘something’ and id =simplified .id);

这种in子查询的形式，在外部表（比如上面的simplified ）数据量较大的时候效率是很差的.

* 数据库本身执行时，都会再把表名给换成自己的别名。
* 主键可以保证记录的唯一和主键域非空,数据库管理系统对于主键自动生成唯一索引，所以主键也是一个特殊的索引。主键列不允许空值，而唯一性索引列允许空值。一个表中可以有多个唯一性索引，但只能有一个主键。
* 根据Mysql内部处理逻辑，使用了别名，就会再建一个临时表放入内存，这样后面的命中会更高。
* 如果将前面的改写，可以放弃子查询和join，提高查询效率

		select b.title from
		(select id from  analysis where 	word=’something’) a, 
		simplified b
		where a.id=b.id;
