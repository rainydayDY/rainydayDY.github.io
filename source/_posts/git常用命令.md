---
title: git常用命令
date: 2017-10-19 20:52:39
tags:
- git命令
categories: 前端
---
### 前言 ###
&emsp;&emsp;git作为版本控制工具，一些基本的命令需要掌握，但是工具只要能够满足需要就行，不能忘记本质，不是搞服务端的不需要研究的很透。
> 此篇为git命令使用的笔记篇

<p hidden><!--more--></p>
### 初次使用 ###
在github上建好仓库后，什么都没添加，仓库名假设为repository。
1. 首先本地建文件夹，拉过来一个文件，初始化git环境
<pre> git init</pre>
2. 添加文件到暂存区(filename是文件名，如果想要添加所有，git add .)
<pre>git add filename</pre>
3. 将文件提交到本地仓库中，describtion是本次提交文件所做的描述
<pre>git commit -m 'describtion'</pre>
4. 首次本地与远程仓库建立连接，如果建立不成功，有可能在此之前，本地的git已经建立过连接，可以删除以前的连接，重新添加(githuburl是你想要连接的仓库的链接地址)
<pre>git remote add origin githuburl</pre>
5. 将本地仓库上传到远程仓库(首次提交的时候 git push -u origin master)
<pre>git push origin master</pre>
删除以往连接
<pre>git remote rm origin</pre>
查看本地git信息
<pre>git config --list</pre>
首次使用时添加用户
<pre>
git config --global user.name 'yourusername'
git config --global user.email 'your register email'
</pre>
查看当前git状态
<pre>
git state
</pre>
