---
layout : post
title : "【备忘】安装hadoop分布式集群时需要配置的环境变量"
category : 工具
tags : [Hadoop]
---

	export JAVA_HOME=/home/hadoop/Applications/jdk1.7.0_45

	export HADOOP_HOME=/home/hadoop/Applications/hadoop-1.2.1
	export HADOOP_HOME_WARN_SUPPRESS=1
	export HADOOP_CONF_DIR=$HADOOP_HOME/conf
	export PIG_HOME=/home/hadoop/Applications/pig-0.11.1
	export PIG_CLASSPATH=$HADOOP_CONF_DIR
	export HIVE_HOME=/home/hadoop/Applications/hive-0.11.0
	export MAHOUT_HOME=/home/hadoop/Applications/mahout-distribution-0.6

	export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$PIG_HOME/bin:$HIVE_HOME/bin:$MAHOUT_HOME/bin