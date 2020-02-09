---
title: 原生js实现轮播图效果
date: 2017-10-19 19:30:33
tags:
- javascript
- 轮播图
categories: 前端
---
### 前言 ###
&emsp;&emsp;实现轮播图效果是我学习js时写的第一段程序，今天算是旧课重温，巩固一下很久不用的原生j语法。

> 定时器的使用，setInterval()  &&  setTimeout()
<p hidden><!--more--></p>
* 实现思路：提到轮播图，一般网站都会用到，pc端的网站通常是放在用户最先看到的地方，每隔几秒钟换一张图片，当所有图片都显示一遍再循环播放，那么很容易想到的就是使用定时器，定时器分为两种:
setTimeout()和setInterval(),二者的区别就是前者是只执行一次，后者是永久执行，除非使用clearInterval()终止，不过如果想要前者拥有和后者同样的效果，可以这样实现：
<pre>
function update(){
  console.log(1);
  var t=setTimeout(update,1000);
}
</pre>
* 注意：虽然实现轮播图js是主要的，css也是功不可没，这里就涉及到父元素和子元素的定位。还有绝对定位如何相对父元素居中的问题，以及not伪类的使用，这里不多说。
*前端的页面结构
<pre>
&lt;div class="container">
    &lt;div class="img_wrap">
        &lt;img src="pictures/1.jpg" alt="加载中...">&lt;/img>
        &lt;img src="pictures/2.jpg" alt="加载中...">&lt;/img>
        &lt;img src="pictures/3.jpg" alt="加载中...">&lt;/img>
        &lt;img src="pictures/4.jpg" alt="加载中...">&lt;/img>
        &lt;img src="pictures/5.jpg" alt="加载中...">&lt;/img>
        &lt;img src="pictures/6.jpg" alt="加载中...">&lt;/img>
    &lt;/div>
    &lt;span class="arrow left_arrow">&lt;&lt;/span>
    &lt;span class="arrow right_arrow">&gt;&lt;/span>
    &lt;div class="dot_line">
        &lt;span class="active">&lt;/span>
        &lt;span class="unselect">&lt;/span>
        &lt;span class="unselect">&lt;/span>
        &lt;span class="unselect">&lt;/span>
        &lt;span class="unselect">&lt;/span>
        &lt;span class="unselect">&lt;/span>
    &lt;/div>
&lt;/div>
</pre>

解释：图片外面要有父元素，因为父元素要设置相对定位，子元素绝对定位，图片的轮播可以通过设置显示或者隐藏，也可以设置opacity。
javascript实现部分：

``` javascript

var dl = document.getElementsByClassName('dot_line');
dl[0].style.marginLeft = -dl[0].offsetWidth / 2 + "px";
var s = dl[0].getElementsByTagName('span');
var par_img = document.getElementsByClassName('img_wrap');
var img = par_img[0].getElementsByTagName('img');
var index = 1,
  t;
for (var i = 0; i < s.length; i++) {
  s[i].index = i;
  s[i].addEventListener('mouseover', function() {
    clearInterval(t);
    quitshow();
    showdot(this.index);
  }, false);
  s[i].addEventListener('mouseout', function() {
    index = this.index + 1;
    t = setInterval(carousel, 2000);
  }, false);
}
function quitshow() {
  for (var j = 0; j < img.length; j++) {
    img[j].style.opacity = 0;
    s[j].className = 'unselect';
  }
}
function showdot(i) {
  img[i].style.opacity = 1;
  s[i].className = 'active';
}
for (var k = 0; k < img.length; k++) {
  img[k].addEventListener('mouseover', function() {
    clearInterval(t);
  }, false);
  img[k].addEventListener('mouseout', function() {
    t = setInterval(carousel, 2000);
  }, false)
}
function carousel() {
  quitshow();
  if (index == 6)
    index = 0;
  showdot(index);
  index++;
}
t = setInterval(carousel, 2000);
var aa = document.getElementsByClassName('arrow');
for (var j = 0; j < aa.length; j++) {
  aa[j].index = j;
  aa[j].addEventListener('mouseover', function() {
    clearInterval(t);
    index--; //因为在定时器里已经加过一次了，现在需要减回来，才能保证传递给点击函数的索引值是当前显示图片的索引
  }, false);
  aa[j].addEventListener('mouseout', function() {
    index++; //因为鼠标移开的时候，此时显示的图片的索引值，还需要等待2秒，在定时器里才能够执行，相当于是4秒后才显示下一张
    t = setInterval(carousel, 2000);
  }, false);
  aa[j].addEventListener('click', function() {
    quitshow();
    if (this.index == 0) {
      if (index == 0)
        index = 6;
      index--;
    } else {
      console.log(index);
      index++;
      if (index == 6)
        index = 0;
    }
    showdot(index);
  }, false);
}

```
实现的效果：图片每隔2秒换一次，下面的焦点也随着显示的图片呈现active状态；鼠标放到图片上或者左右箭头或者焦点上，图片都会停止轮播；鼠标移开，继续轮播，鼠标移到某个焦点上，就会显示对应的图片。
思路描述：首先考虑图片能够自动播放，定时器函数，就是让index自增，当增加到数组长度时，令index为0，这是一个循环，然后重复这个过程，使用setInterval，在让图片显示的过程中，那么一定要让其他图片隐藏，所以，每次执行carousel函数之前都要执行一次所有图片隐藏的函数，接下来考虑焦点的触摸显示对应图片，也存在当前焦点处于active状态，其他焦点都处于unselect状态，所以抽离出一个循环函数。上一张下一张的也同理，不同的是，在查看下一张的时候要先自增，再判断，而查看上一张的时候先判断再自减。
其实如果在开发中真正需要用轮播图，完全可以用现成的插件来做，他们可扩展性更高，你只要写好html就好了，比如swiper，比如owl.carousel，我只用过这两个，觉得挺好用的，毕竟在公司中开发很注重效率，但是你也一定要会自己实现，这样你去改别人的代码的时候就不会无所适从了。
结束。
