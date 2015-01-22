---
layout : post
title : "一些有用的grep命令"
category : 工具
tags : [Linux]
---

1. 在某个文件中搜索指定字符串

	在/etc/passwd文件中，搜索字符串guolei：
	
		grep 'guolei' /etc/passwd
	
	注意，guolei的引号可以省略，但是如果搜索字符串中有空格或者你使用正则表达式，就需要加了。
	
	<!--more-->

2. 在多个文件中搜索指定字符串

	在当前目录中，搜索包含字符串guolei的文件：

		grep -r guolei *

	注意：-r是recursive的缩写，表示递归的搜索。

	在当前目录的.java文件中，搜索包含字符串guolei的文件：

		grep -r guolei *.java
	
	有时候，我们的搜索结果可能比较多，我们可以结合less命令来展示结果：
	
		grep -r guolei *.java | less
	
	或者搜索结果比较多，我们只需要列出文件名：

		grep -rl guolei *.java
		
	还有一种需求比较常见，我们经常想找到某一个目录中，包含指定字符串的文件。比如，我们想在当前目录下递归的查找所有.java文件中包含字符串guolei的文件：

		find . -type f -name *.java -exec grep -il guolei {} \;
		
3. 搜索时忽略大小写

	在搜索guolei时，忽略大小写：

		grep -ri guolei *
	
	注意：-i是Ignore case的缩写，表示忽略大小写。

4. 搜索结果中列出行号

	在搜索结果中，列出字符串出现位置的行号码：
	
		grep -rn guolei *.java

	注意-n是number的缩写，表示行号的意思。

5. 反向搜索

	实际开发中，还有一种情况比较常见，我们要在某个目录下搜索不包含某个字符串的文件：

		grep -riv guolei * | less

	注意：-v是reverse的缩写，表示逆向的意思。上面的例子为在当前目录中搜索不包含guolei的文件。

6. 在管道中使用grep

	我们经常还会在管道命令中使用grep，这个最常见。比如我们要搜索目前系统中的mysql进程：

		ps -ef | grep mysql

	或者列出当前目录以html结尾的文件：

		ls | grep 'html$'