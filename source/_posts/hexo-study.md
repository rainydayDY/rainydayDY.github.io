---
title: HEXO 的使用
date: 2017-08-16 11:57:47
tags:
- 创建博客
- 创建文章
- 命令使用
categories: 前端
---
### 前言 ###
&emsp;&emsp;第一次接触hexo，命令总是忘，用markdown写文章也总是忘记都该写什么，所以决定专门写一篇笔记。<br>
> 勤能补拙，养成每天学新知的习惯。

<p hidden><!--more--></p>


* 博客初始化：
    <pre>$ hexo init [floder]</pre>(floder是自己定义的文件名)
* 本地浏览器查看博客
    <pre>$ hexo server</pre>
然后在地址栏中输入localhost:4000即可查看
* 写文章
    <pre>$ hexo new "文章名称"</pre>
    source/_posts下会生成一个“文章名称.md”的文件，打开即可编辑。
*  tags是标签的名称，可以自定义，书写格式如下:
> -&emsp;HTML
> -&emsp;CSS
> -&emsp;性能优化
* categories是类别名称，直接在冒号后面写即可
* 空格的使用“&amp;emsp;&amp;emsp;”
*第一次将内容部署到github上
<pre>cnpm install hexo-deployer-git --save<br></pre>
* 上传文章
<pre>
$ hexo clean<br>
$ hexo g<br>
$ hexo d
</pre>
<table>
<tr><th>选项</th><th>描述</th></tr>
<tr><td>clean</td><td>清除缓存文件 (db.json) 和已生成的静态文件 (public)。</td></tr>
<tr><td>d,--deployer</td><td>文件生成后立即部署网站</td></tr>
<tr><td>g,--generate</td><td>部署之前预先生成静态文件</td></tr>
</table>
