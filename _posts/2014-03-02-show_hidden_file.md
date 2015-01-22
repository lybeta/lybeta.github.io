---
layout : post
title : "设置Mac下显示/隐藏文件快速切换 "
category : 工具
tags : [Mac]
---

打开Automator（应用程序文件夹中）并选择服务。在资源库中选中“Run Shell Script”并将它拖到右边的工作区中。将以下代码复制到文本框中。 

	STATUS=`defaults read com.apple.finder AppleShowAllFiles` 
	if [ $STATUS == YES ]; 
	then 
	defaults write com.apple.finder AppleShowAllFiles NO 
	else 
	defaults write com.apple.finder AppleShowAllFiles YES 
	fi 
	killall Finder 
	
最后在上边的Service receives的下拉菜单中选择“no input”，然后将工作流程保存为“Toggle Hidden Files”。 

<!--more-->

现在，如果你打开Finder——服务菜单，你会看到刚才制作的“Toggle Hidden Files”选项。现在添加键盘快捷键，打开系统偏好设置——键盘，点击快捷键选项卡。在左边选择服务，然后勾上“Toggle Hidden Files”，在它的右边双击鼠标，然后按下你想要设定成为的快捷键。我使用的是Command+Shift+.（点）。 

设置一个右键菜单项 

与以上设置快捷键中使用Automator的过程相同，唯一的区别在于，上边提到的那个Service receives，这次选中“文件和文件夹”，并在右边选中Finder。保存。现在，当你在finder中弹出右键菜单时，就会看到“Toggle Hidden Files”选项咯。