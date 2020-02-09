---
title: webview相关问题
date: 2019-03-15 16:22:18
tags:
- 软键盘遮挡
- 使用场景
categories: 前端
---
> 解决webview下软键盘弹出遮挡输入框的问题，简单理解webview的使用场景
<p hidden><!--more--></p>

## webview下软键盘弹出遮挡输入框
### 场景描述
正常场景下，软键盘弹出，会将输入框推到软键盘上方，这样用户可以一边输入，一边看到自己已经输入的内容，然而在webview下，软键盘的弹出是由webview内的网页元素触发的，但是软键盘又不属于网页元素，这样软键盘就会遮挡输入框。


### 尝试方案
手动触发页面滚动，如果输入框恰好在页面最底端，那就加占位元素，使得在获取焦点的时候，占位元素显示，将输入框顶到软键盘上方，然而，在安卓机上，失去焦点的时候，软键盘并不会收起，这样就没办法找到时机将占位元素隐藏，安卓手机（部分机型）的软键盘的收起需要手动触发。
于是，就一直想着监听软键盘的收起弹出操作，然后加以控制。
``` javascript 
    componentDidMount() {
        this.clientHeight = document.documentElement.clientHeight || document.body.clientHeight;
        window.addEventListener('resize', this.resize);
    }
    resize() {
        const nowClientHeight = document.documentElement.clientHeight || document.body.clientHeight;
        if (this.clientHeight > nowClientHeight) {
            alert('键盘弹出');
        } else {
            alert('键盘收起');
        }
    }
```
以上代码，如果用手机浏览器调试，是没有问题的，会按照预期的弹出键盘和收起键盘的时候分别给出提示，然而，在webview下竟然连resize事件都不触发了，试了各种关键词搜索，都没有找到答案，关键时刻还是stackoverflow比较靠谱，他可以根据你的词汇，找出所有相关方向的问题，整个过程中我搜的关键词都是“如何监听键盘的收起” “webview下resize无效” “webview下监听键盘收起”，都没有找到预期答案，stackoverflow给的问题方向全部都是android方向的，于是按照此方向查，找到答案，又去找移动端同学帮忙加上，才解决了问题。


### webview下不触发resize的原因
webview的相关布局被固定了高度，这里也分为两种情况：<br>
    a) 根布局固定了高度，这里的根布局是webview所在的Activity的最外层布局；<br>
    b) 根布局未固定高度，但是根布局是FrameLayout布局，而webview或者其父控件被固定了高度。


### 解决方案
a ：非全屏模式：<br>
状态栏是app的，只有下面的content是我们的，这种情况下很好解决，将windowSoftInputMode设置为adjustResize即可<br>
b：全屏模式：<br>
引入AndroidBug5497Workaround类，在创建webview的时候添加一句AndroidBug5497Workaround.assistActivity(this)即可。

## webview使用场景

1. 使用loadUrl显示链接：比如我们在qq或者微信中打开一个链接，qq解析出这个文本是一个网址时就通过webview去加载它。
2. app内某个模块完全用h5实现，app需要提供一个入口，此时webview相当于一个编译容器，用来显示整个模块的内容，模块内可以有自己的整套逻辑。比方说淘宝，可能整个首页都是webview里面的内容。


### 参考文章
1. [Android 爬坑之旅：软键盘挡住输入框问题的终极解决方案](https://www.diycode.cc/topics/383)
2. [深入讲解WebView——上](https://www.kymjs.com/code/2015/05/03/01/)
3. [深入讲解WebView——下](https://kymjs.com/code/2015/05/04/01/)







