---
title: ES6学习笔记
date: 2017-11-13 13:44:20
tags:
- javascript
- ES6
categories: 前端
---

<p hidden><!--more--></p>
## let 和 const命令 ##
1.let命令只在变量声明的代码块内有用。（块级作用域）
```javascript
var a=[];
for(let i=0;i<10;i++){
	a[i]=()=>{
		console.log(i)
	}
}
a[6]();
//执行结果为6,每一轮循环的变量i的值都是重新声明的，但是js引擎在初始化本轮的变量i时，会在上一轮循环的基础上进行运算
```
2.不存在变量提升（变量一定要在声明后使用）
3.暂时性死区（只要进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有声明后才可以获取和使用）
4.块级作用域：外层作用域无法访问内层作用域的变量。
5.do表达式，使得块级作用域能够有返回值。
```javascript
let a=do{
	let t=f();
	t=t*t+1;
}
```
6.const声明常量，比较安全的行为是声明数值、字符串、布尔值。尽量不要去声明数组和对象。
7.var和function声明的全局变量，依旧是顶层对象的属性，另一方便，let/const/class声明的全局变量，不属于顶层对象的属性。
## 字符串的扩展
1.测试一个字符是由两个字节组成还是4个（判断英文和汉字）：codePointAt()
2.遍历字符串（for ... of）
```javascript
let text="hello";
for(let i of text){
	console.log(i)
}
```
3.确定一个字符串是否包含在另一个字符串中。
> includes()  表示是否找到了字符串
> startsWith()  表示参数字符串是否在原字符串的头部
> endsWith()  表示参数字符串是否在原字符串的尾部
返回布尔值

4.repeat()方法返回一个新的字符串，表示将原字符串重复n次
5.template string（模板字符串）

typeof 安全防范机制
