---
title: babel学习
date: 2019-09-25 20:05:50
tags:
---


babel:
1. 语法转换
2. 通过polyfill的方式在目标环境中添加缺失的特性
3. 源码转换
原理：通过语法转换器来支持新版本的javascript语法

react项目：
@babel/preset-react  @babel/core  @babel/preset-env

babel的核心功能 @babel/core 
polyfill：在原型上添加方法
useBuiltIns: "usage"，打包时只打包所需要使用的polyfill

babel配置文件
babel.config.js  项目范围的配置，项目根目录
.babelrc 文件范围的配置
