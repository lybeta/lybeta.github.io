---
layout : post
title : "在 Ubuntu 虚拟机中使用命令行安装 VMware Tools"
category : 工具
tags : [Linux]
---

1. 转到虚拟机 > 安装 VMware Tools（或 VM > 安装 VMware Tools）。
注意：如果您运行的是轻量版的 Fusion、不带 VMware Tools 的 Workstation 版本或 VMware Player，则系统会提示您先下载 Tools，然后才能安装它们。此时请单击立即下载开始进行下载。

	<!--more-->

2. 在 Ubuntu 客户机中，运行以下命令：
	
		sudo mkdir /mnt/cdrom
	
	当系统提示输入密码时，请输入 Ubuntu 管理员用户密码。

	注意：出于安全考虑，所键入的密码不会显示出来。在接下来的五分钟内您无需重新输入您的密码.
	
		sudo mount /dev/cdrom /mnt/cdrom 或 sudo mount /dev/sr0 /mnt/cdrom

	VMware Tools 捆绑包的文件名因您的 VMware 产品版本而异。运行下面的命令可查找确切的名称：
	
		ls /mnt/cdrom
		tar xzvf /mnt/cdrom/VMwareTools-x.x.x-xxxx.tar.gz -C /tmp/

	注意：x.x.x-xxxx 是在上一步中查明的版本。
		
		cd /tmp/vmware-tools-distrib/	
		sudo ./vmware-install.pl -d

	注意：-d 开关假定您希望接受默认设置。如果不使用 -d，请按 Return 接受各个默认值或提供您自己的答案。

3. 安装完成后运行下面的命令重新启动虚拟机：

		sudo reboot