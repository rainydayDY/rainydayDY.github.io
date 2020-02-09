---
title: CSS笔记
date: 2017-09-15 14:46:59
tags:
- CSS
- HTML
categories: 前端
---
#### 前言 ####

&emsp;&emsp;虽然从大二就开始接触CSS，但是由于CSS细节方面需要学习的很多，所以还是需要不断的探索，此篇大概会一直处于未完待续的状态，因为我要边学边记。
> 路漫漫其修远兮，吾将上下而求索

<p hidden><!--more--></p>
### September 15th ###
* css box-sizing

&emsp;&emsp;CSS border-box的用法：（我以前看到定义了宽高和内边距，内边距向里延伸，盒子实际宽高不变的现象都是ie的bug呢，直到今天尝试了border-box才发现，原来不是啊），一般情况下，元素如果定义了：
```css
width:100px;
height:100px;
padding:20px;
```
&emsp;&emsp;那么元素的实际宽高就是140px的，这是现在浏览器的计算方式。content的部分是不包括padding和border的。而ie中的content是包括这两者的，在ie中，你这样定义元素，那么实际宽高就是100px，如果你将元素定义了:
```css
box-sizing: border-box;
```
那么元素的宽高就是100px。
<br>
* 元素未定义宽高，定义margin或者padding

&emsp;&emsp;如果一个元素，未定义宽高，设置了margin 或者padding，他就会以block的形式显示，宽高默认和父元素的一致；（确切的说是和最近的容器容量宽高一致，比如在上面的元素里再定义一个div，如果设置里padding为10px，那么她的宽就是100px,高度是120px；
<br>

### "iconfont"的使用方式（html画字体图标）###
* <a href="http://www.iconfont.cn/">阿里巴巴矢量图标库</a>
&emsp;&emsp;仿佛发现了新大陆，第一次使用iconfont也是无意中发现这个东西，后来就每次使用都下载，觉得很方便，因为去切图真的是麻烦，尤其是对我这种视力不好的人，用了大半年发现自己真的是low爆了，今天又开始模仿，看了别人的源码发现原来可以直接引用啊。

1、将你需要的icon加入购物车（加入暂存架）
2、选择完所有的图标后“存储为项目”，然后下载到本地，有四种格式的图片，分别是.eot/.woff/.ttf/.svg，使用的时候将这四个文件存为font，再按照@font-face的方式引用：
```css
@font-face {
  font-family: "iconfont";
  src: url('iconfont.eot'); /* IE9*/
  src: url('iconfont.eot#iefix') format('embedded-opentype'), /* IE6-IE8 */
  url('iconfont.woff') format('woff'), /* chrome, firefox */
  url('iconfont.ttf') format('truetype'), /* chrome, firefox, opera, Safari, Android, iOS 4.2+*/
  url('iconfont.svg#iconfont') format('svg'); /* iOS 4.1- */
}
.iconfont {
  font-family:"iconfont" !important;
  font-size:16px;
  font-style:normal;
  -webkit-font-smoothing: antialiased;
  -webkit-text-stroke-width: 0.2px;
  -moz-osx-font-smoothing: grayscale;
}
```
其实在另一方面，这也是引用外部字体的方式啊，设计师总喜欢用苹方体。但是通常他们只给提供ttf格式的文件，IE是没办法引用这种格式的字体文件的，网上有很多转换工具，转为eot格式的即可使用。
```css
@font-face {
  font-family: 'PingFang';
  src:url(./fonts/pingfang.eot?#iefix) format('embedded-opentype'), url("./fonts/pingfang.ttf") format('truetype');
}
*{
  font-family: PingFang;
}
```
还有一种情况，设计师大多数情况上传的svg都是带有颜色的，这时候你去以字体的方式为其添加颜色，会不生效，没有关系，直接修改svg的代码，修改title为你想要的值，将fill置为空，当然，这并不适用于所有的，不能将所有的fill都置为空，按照需求来。
3、在html中使用类名
<pre>
&lt;i class="iconfont">&#59053;&lt;/i>
</pre>
### css 常用属性 ###
1. 修饰文本：text-transform:uppercase/capitalize(文本中的每个单词以大写字母开头)/lowercase
2. 首行缩进：text-indent:10px(不仅可以用在文本中，也可以用在布局中，如联系方式在右上角，还带图标的，可以这样做)
3. 字间距：letter-spacing:5px
4. 单词间的空格：word-spacing:5px
5. 文本不换行：white-space:nowrap;
6. 表格边框去除空隙:border-collapse:collapse
7. outline:none;  清除外边框

### css 布局 ###
1. 实现上三角或者下三角，用border。
2. 在实现的过程中，考虑结构与表现的分离，尽可能少用标签来组成页面，根据所需元素选择标签，然后再去想样式，而不应该是先看见效果图，想布局，然后再添加元素。比如常见的朋友圈列表，头像在左面，帖子在右面，一般人会想到分为两部分，用浮动布局，但实际上可以考虑由一部分组成，头像设置margin-left为负值。
3.


### 项目中遇见问题 ###
1. 手机端点击文字时出现背景色(浏览器上慢速操作按下和弹起这个操作也会出现这个问题，需要把触摸时的高亮设置为透明色即可)
解决方案：
```css
.left_tab>li a{
  background-color: transparent !important;
  -webkit-tap-highlight-color:transparent;
}
```
2. 隐藏overflow:scroll时的和滚动条，可以设置滚动条宽度为0，设置display:none，浏览器上有效果，但是移动端无效。
```css
.left_tab::-webkit-scrollbar{
  /*display: none;*/
  width: 0
}
```
3. IOS中对overflow:scroll出现的卡顿
```css
.left_tab{
    overflow: scroll;
    -webkit-overflow-scrolling:touch;
}
```
4. 手机端用rem适配后，得到的值可能出现小数，而jquery中的scrollTop()得到的值是整数，即使使用锚点定位当前的scrollTop为小数，也会进行四舍五入，所以在获取高度的时候就要使用Math.round()，这样就能匹配正确。
描述的不太清楚，我直接说业务需求，左侧tab，右侧内容，包含所有内容，右侧内容在滚动的时候，左侧的tab，要对应显示active和usual，左侧tab点击的时候，右侧要滚动到合适的位置。
