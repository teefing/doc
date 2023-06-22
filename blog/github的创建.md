---
title: github的创建
date: 2017-4-2
tags: 
	- github
category: 专业知识
---
# github的创建

* 进入github帐号后，点击右上角+号，点击new repository，库的名称可任意，但是只有名称为“用户名.github.com”的名称创建的git可以通过外部网络访问
* 安装Git，从官网下载适合自己电脑的Git，点击安装，一路“Next”就可以了。安装完成，打开Console开始设置Git参数。

        git config --global user.name "xxx"  
        git config --global user.email "xxx@xxx.xxx"
        //在上面的两个引号中分别填写你的名字和邮箱。
        //由于Git是分布式的版本控制系统，可能会有很多用户，每个用户需要有自己的名字和邮箱来互相区分。
设置完毕后，可以通过命令行`git config -l`查看全局配置信息

* cmd进入选择作为版本库的文件夹
* 输入命令

		git init
		git add .//会把该目录内的所有文件复制到暂存区
		git commit -m "备注"
		git remote add origin https://github.com/用户名/版本库名.git
		git push -u origin master
* 再进入github中你的版本库，发现东西都已经上传了


