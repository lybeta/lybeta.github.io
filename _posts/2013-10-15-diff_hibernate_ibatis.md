---
layout: post
title: "hibernate和ibatis区别"
category: 技术
tags: [Java]
---

hibernate 是当前最流行的o/r mapping框架，它出身于sf.net，现在已经成为jboss的一部分了。

ibatis 是另外一种优秀的o/r mapping框架，目前属于apache的一个子项目了。 相对hibernate“o/r”而言，ibatis是一种“sql mapping”的orm实现。 
     
hibernate对数据库结构提供了较为完整的封装，hibernate的o/r mapping实现了pojo 和数据库表之间的映射，以及sql 的自动生成和执行。程序员往往只需定义好了pojo 到数据库表的映射关系，即可通过hibernate 提供的方法完成持久层操作。程序员甚至不需要对sql 的熟练掌握， hibernate/ojb 会根据制定的存储逻辑，自动生成对应的sql 并调用jdbc 接口加以执行。 

而ibatis 的着力点，则在于pojo 与sql之间的映射关系。也就是说，ibatis并不会为程序员在运行期自动生成sql执行。具体的sql 需要程序员编写，然后通过映射配置文件，将sql所需的参数，以及返回的结果字段映射到指定pojo。 使用ibatis 提供的orm机制，对业务逻辑实现人员而言，面对的是纯粹的java对象。这一层与通过hibernate 实现orm 而言基本一致，而对于具体的数据操作，hibernate会自动生成sql 语句，而ibatis 则要求开发者编写具体的sql 语句。相对hibernate而言，ibatis 以sql开发的工作量和数据库移植性上的让步，为系统设计提供了更大的自由空间。 

<!--more-->

**hibernate与ibatis的对比：**

1. ibatis非常简单易学，hibernate相对较复杂，门槛较高。
2. 二者都是比较优秀的开源产品
3. 当系统属于二次开发,无法对数据库结构做到控制和修改,那ibatis的灵活性将比hibernate更适合
4. 系统数据处理量巨大，性能要求极为苛刻，这往往意味着我们必须通过经过高度优化的sql语句（或存储过程）才能达到系统性能设计指标。在这种情况下ibatis会有更好的可控性和表现。
5. ibatis需要手写sql语句，也可以生成一部分，hibernate则基本上可以自动生成，偶尔会写一些hql。同样的需求,ibatis的工作量比hibernate要大很多。类似的，如果涉及到数据库字段的修改，hibernate修改的地方很少，而ibatis要把那些sql mapping的地方一一修改。
6. 以数据库字段一一对应映射得到的po和hibernte这种对象化映射得到的po是截然不同的，本质区别在于这种po是扁平化的，不像hibernate映射的po是可以表达立体的对象继承，聚合等等关系的，这将会直接影响到你的整个软件系统的设计思路。
7. hibernate现在已经是主流o/r mapping框架，从文档的丰富性，产品的完善性，版本的开发速度都要强于ibatis。