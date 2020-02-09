---
title: webpack 学习笔记
date: 2017-11-14 13:53:30
tags:
- javascript
- 模块化
categories: 前端
---
> 目标：使用webpack搭建一个脚手架

<p hidden><!--more--></p>
### 概念###
本质上：模块打包器（module bundler）,构建依赖关系图，将所有模块打包成一个或者多个bundler。
核心概念：
1、entry(入口起点：单个和多个)
```javascript
module.exports={
	entry:__dirname+'/app/main.js' //一个入口文件
	entry:{
		pageone:__dirname+'/app/main1.js',
		pagetwo:__dirname+'/app/main2.js'
		//多个入口文件
	}      
}
```
> 经验：每个HTML文档只使用一个入口起点

2、output（出口：打包的输出目录和输出文件的名字）
```javascript
module.exports={
	output:{
		path:__dirname+'/public',
		filename:"bundle.js" //当有多个入口起点的时候，对于输出文件应该使用占位符来确保每个文件具有唯一的名称。
		filename：‘[name].js’
	}
}
```
> 即使可以存在多个入口起点，但只指定一个输出配置

3、loader(使得webpack可以额外去处理那些非javascript文件，因为本质上webpack自身只理解javascript，loader可以将所有类型的文件转换为有效的模块，使用rules属性，必须包含test和use这两个属性，test为规则,use为转换工具)
常用的loader：css-loader、style-loader、babel-core、babel-loader、babel-preset-es2015、babel-preset-react，其中前两个是处理
css的，后面的是转换js的，用loader前需要安装
```javascript
npm install --save-dev babel-core babel-loader babel-preset-2015 babel-preset-react
npm install --save-dev style-loader css-loader
//配置文件中要指明匹配的正则表达式和使用的loader
module.exports={
	module:{
		rules:[
			{
				test:/(\.jsx|\.js)$/,
				use:'babel-loader'
			},
			{
				test:/\.css$/,
				use:[
					{
						loader:'style-loader'
					},{
						loader:'css-loader'
					}
				]
			}
		]
	}
}
```
> loader的使用方式有三种，具体可以参考官网

4、plugins（插件：打包优化压缩，定义环境变量等等）
使用方法：先require()，再把它添加到plugins数组中，具体参照官网
### 使用 ###
1、安装
<pre>
npm install -g webpack   //全局安装
</pre>
2、新建一个文件夹，（以下操作都是基于这个文件夹的）初始化一个webpack项目
<pre>
npm init
npm install --save-dev webpack //在本项目中安装webpack依赖包
</pre>
3、新建app和public两个文件夹：app存放的是原始数据，原始文件，public存放的是供浏览器读取的文件，即编译后的文件。

4、新建webpack.config.js文件，为配置文件。
> 建在项目的根目录下

5、修改package.json文件的scripts属性，新增
<pre>
"scripts":{
	"start":"webpack"
}      //package.json不支持注释
</pre>
6、生成source maps(便于调试的，对于中小型项目，使用eval-source-map即可)
```javascript
module.exports={
	devtool:'eval-source-map'
}
```
> source map内容在buddle.js里面
7、构建本地服务器（浏览器监听代码修改，并自动刷新显示修改后的结果）
<pre>
npm install --save-dev webpack-dev-server
</pre>
配置webpack.config.js文件
```javascript
module.exports={
	devServer:{
		contentBase:'./public',//本地服务器所加载的页面所在的目录
		historyApiFallback:true,//所有的跳转都指向index.html，即不跳转
		inline:true //实时刷新
	}
}
```
在package.json中配置scripts用以开启本地服务器
<pre>
"scripts":{
	"start":"webpack",
	"server":"webpack-dev-server --open"
}     
</pre>

8、结束
<pre>
npm start //打包
npm run server //启动
</pre>
