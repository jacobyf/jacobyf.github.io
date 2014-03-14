---
layout: post
title: "Nginx Windows安装注意的事情"
date: 2014-03-07 06:36:00
categories: [coding, skills]
---

nginx的Windows版本使用原生Win32 API（非Cygwin模拟层）。当前nginx/Windows只使用select作为通知方法，所以不要期待它有很高的性能和扩展性。鉴于这点和一些已知问题，nginx/Windows目前还处于beta阶段。nginx/Windows和Unix版本相比，功能几乎已经齐全，除了XSLT过滤器、图像过滤器、GeoIP模块和嵌入Perl语言支持以外。

安装nginx/Windows，需要下载最新的1.5.11开发版本，因为开发分支上包含了所有已知的问题修复，尤其是针对Windows版本的问题修复。解压缩下载得到的zip文件，进入nginx-1.5.11目录，运行nginx。下面给出一个在C盘根目录下安装的例子：

		cd c:\
		unzip nginx-1.5.11.zip
		cd nginx-1.5.11
		start nginx
可以在命令行运行tasklist命令来查看nginx进程：

		C:\nginx-1.5.11>tasklist /fi "imagename eq nginx.exe"

		Image Name           PID Session Name     Session#    Mem Usage
		=============== ======== ============== ========== ============
		nginx.exe            652 Console                 0      2 780 K
		nginx.exe           1332 Console                 0      3 112 K

nginx/Windows作为标准控制台应用运行，而不是系统服务。可以用下面的命令控制：

		nginx -s stop	快速退出
		nginx -s quit	优雅退出
		nginx -s reload	 更换配置，启动新的工作进程，优雅的关闭以往的工作进程
		nginx -s reopen	重新打开日志文件