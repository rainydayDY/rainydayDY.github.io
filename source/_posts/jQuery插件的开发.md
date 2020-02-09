---
title: jQuery插件的开发
date: 2017-10-24 15:10:16
tags:
- 插件
- jQuery
categories: 前端
---
### 前言 ###
&emsp;&emsp;参考链接：<a href="http://www.jianshu.com/p/518d424d4994">掌握jQuery插件开发，这篇文章就够了</a>
<p hidden><!--more--></p>
&emsp;&emsp;在实际开发工作中，总会碰到像滚动，分页，日历等展示效果的业务需求，对于接触过jQuery以及熟悉jQuery使用的人来说，首先想到的肯定是寻找现有的jQuery插件来满足相应的展示需求。目前页面中常用的一些组件，都有多种jQuery插件可供选择，网络上也有很多专门收集jQuery插件的网站。利用jQuery插件确实可以给我们的开发工作带来便捷，但是如果只是会简单使用，而对其中的原理不甚了解，那么在使用过程中碰到问题或者对插件进行定制开发时就会有诸多疑惑。本文的目的就是可以快速了解jQuery插件的开发原理以及掌握jQuery开发的基本技能。

&emsp;&emsp;进行jQuery插件开发前，首先要知道两个问题：
1. 什么是jQuery插件？
2. jQuery插件如何使用？
第一个问题，jQuery插件就是用来扩展jQuery原型对象的一个方法，简单来说就是jQuery插件是jQuery对象的一个方法。其实回答了第一个问题，也就知道第二个问题的答案了，jQuery插件的使用方式就是jQuery对象方法的调用。

我们先看个例子：$("a").css("color","red")。我们知道每个jQuery对象都会包含jQuery中定义的DOM操作方法,这里使用$方法来选择a元素，返回一个a元素的jQuery对象，这个对象就可以使用jQuery中定义的DOM操作方法。那么jQuery对象是如何获取这些方法的呢？其实jQuery内部定义了一个jQuery.fn对象，查看jQuery源码可以发现jQuery.fn=jQuery.prototype，也就是说jQuery.fn对象是jQuery的原型对象,jQuery的DOM操作方法都在jQuery.fn对象上定义的，然后jQuery对象就可以通过原型继承这些方法。

* 基础版jQuery插件
&emsp;&emsp;知道了上面这些知识，我们就可以来写一个简单的jQuery插件。假如我现在需要一个jQuery插件用来改变标签内容颜色，就可以按下面的方式来实现这个插件：
``` javascript
$.fn.changeStyle = function(colorStr){
         this.css("color",colorStr);
}
```
然后按下面的方式来使用插件：
```javascript
$("p").changeStyle("red");
```
&emsp;&emsp;插件调用的时候，插件内部的this就是当前调用插件的jQuery对象，这样的话每个使用$()方法选择的标签，在调用changeStyle()插件时都会使用css()方法重设color样式。

* 满足链式调用的jQuery插件
链式调用时jQuery的一大特色，一个通用的插件应该遵循jQuery风格，满足链式调用要求。实现链式调用的方式也很简单：
```javascript
$.fn.changeStyle = function(colorStr){
         this.css("color",colorStr);
         return this;
}
```
然后使用的时候就可以链式调用其他方法了：
```javascript
$("p").changeStyle("red").addClass("red-color");
```
实现链式调用的关键点就一行代码return this，插件中加了这行代码，那么在插件执行完之后，就会把当前的jQuery对象返回，然后就可以在插件方法后面继续调用其它jQuery方法。

防止$符号污染的jQuery插件
有很多js库都会使用$符号，虽然jQuery可以使用jQuery.noConflict()方法交出$符号的使用权，但是如果定义插件的时候，使用$.fn对象来定义的，那么这些插件使用的时候就会受到其它使用$变量的js库的影响。对于这种情况，我们可以使用立即执行函数通过传参的方式封装插件。形式如下：
```javascript
(function($){
     $.fn.changeStyle = function(colorStr){
         this.css("color",colorStr);        
         return this;
     }
}(jQuery));
```
因为使用了立即执行函数，所以此时的$只属于这个立即执行函数的函数作用域，这样就可以避免$符号的污染。

* 可以接受参数的jQuery插件
继续上面的例子，假如我还想为这个插件添加一个设置标签元素内容文字大小的功能，那么我可以这么来实现：
```javascript
(function($){
     $.fn.changeStyle = function(colorStr，fontSize){
         this.css("color",colorStr).css("fontSize",fontSize+"px");        
         return this;
     }
}(jQuery));
```
上面这种插件传参方式适用于参数比较少的情况，如果需要传给插件内部的参数比较多，我们可以定义一个参数对象，然后把需要传给插件的参数放在参数对象中。插件定义时如下：
```javascript
(function($){
     $.fn.changeStyle = function(option){
         this.css("color",option.colorStr).css("fontSize",option.fontSize+"px");        
         return this;
     }

}(jQuery));
```
使用方式：
```javascript
$("p").changeStyle({colorStr:"red",fontSize:14});
```
把参数放到一个对象中传给插件还有一个好处就是我们可以在插件内部为一些参数定义一些缺省值，例如：
```javascript
(function($){

     $.fn.changeStyle = function(option){
          var defaultSetting = { colorStr:"green",fontSize:12};
          var setting = $.extend(defaultSetting,option);
          this.css("color",setting.colorStr).css("fontSize",setting.fontSize+"px");        
         return this;
     }
}(jQuery));
```
&emsp;&emsp;上面的代码用到了$.extend方法，这个方法在这里的用法就是合并两个对象，即把后面一个对象的存在的属性值赋值给第一个对象，具体用法可以参考这里。$.extend方法还有一种作用是用来扩展jQuery对象本身。
这样定义的插件，我们在使用时如果不传fontSize，那么使用这个插件的jQuery对象标签的内容会被设置成默认的12px。
使用方式：$("p").changeStyle({colorStr:"red"});
注意：在为插件定义默认参数时，一定要把默认参数写在插件方法内部，这样默认参数的作用域就在插件内部。

* 总结
定义插件的方式除了上面说的用$.fn来定义，还有另外一种方式来定义插件，那就是使用$.fn.extend方法。类似下面的写法：
```javascript
(function($){
     $.fn.extend({         
         changeStyle:function(option){             
         var defaultSetting = { colorStr:"green",fontSize:12};
         var setting = $.extend(defaultSetting,option);
         this.css("color",setting.colorStr).css("fontSize",setting.fontSize+"px");        
         return this;
          }
     });
}(jQuery));
```
PS:$.extend方法和$.fn.extend方法都可以用来扩展jQuery功能，通过阅读jQuery源码我们可以发现这两个方法的本质区别，那就是$.extend方法是在jQuery全局对象上扩展方法，$.fn.extend方法是在$选择符选择的jQuery对象上扩展方法。所以扩展jQuery的公共方法一般用$.extend方法，定义插件一般用$.fn.extend方法。
实现的例子：<a href="https://github.com/rainydayDY/market/tree/master/multiselect">jQuery插件实现条件多选</a>
