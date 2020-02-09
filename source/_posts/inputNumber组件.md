---
title: inputNumber组件
date: 2018-12-06 16:38:57
tags:
- input number
- 兼容
- react
categories: 前端
---
> 需求描述：实现只能输入一位正小数的输入框，且在移动端点击，弹出数字键盘。
<p hidden><!--more--></p>

## 实现效果
1. 数字输入框，前端严格限制并且验证用户的输入行为，不得输入非法字符，输入非法字符，就清空之前输入的数字；
2. 小数点后只能输入一位数字，不得输入负数；
3. 移动端唤起数字键盘
4. 限制整数位最多输入7位
## 思路分析
1. 数字类型，使用原生的input输入框，设置type为number或者tel；
2. 绑定事件onchange，用户每输入一个就判断一次，用正则过滤；
3. substr做字符串截取;
***方案1***

实现如下：

```javascript
 const { value } = this.props;
 const val = value ? parseFloat(value) : value; // 去掉首部0
<input value={val} placeholder="0.0" type="number"  onChange={this.handleChange}/>
handleChange = (e) => {
    e && e.preventdefault && e.preventdefault();
    const { onChange } = this.props;
    let val = e.target.value;
    val = String(val).replace(/^(-)*(\d+)\.(\d).*$/, '$1$2.$3');
    val = val.substr(0, 7);
    onChange(val >= 0 ? val : ''); 
}
```
顺着这个思路做，是没有问题的，输入小数，小数点后面就只能输入一位，并且只可以输入数字，但是，多测几次，问题就来了，设置type=”number”，字母和标点符号都不能输入，诸如’e’，’+’，’-‘，’.’这样的字符是可以无限输入的，而且输入的这些非法字符都无法获取到，既然无法获取到，也就无法做过滤，无法做截取，当输入了非法的字符，获取到的值是’’，所以可以根据这个值，做一些限制，比如说可以在每次检测到val变为’’的时候，不去设置这个值，而是采用上次的值：
***方案2***

```javascript
const { value } = this.props;
let val = e.target.value;
val = val === '' ? value : val;
```
这样的处理又会带来新的问题，因为不管是用户自身触发了输入非法字符，还是进行了清空操作，都会导致获得的val为’’，这样的结果就是一直都不能清空，上一次输入了31.3…，处理之后值为31.3，删除后，会一直都有3。。显然这样是不允许的。如果改为下面这样:
***方案3***
```javascript
val = val === '' ? '' : val;
```
结果就是不生效，设置为空，非法字符就还会存在，除非设置一个和’’不一样的值，那就设置’0’吧，
***方案4***

```javascript
val = val === '' ? '0' : val;
```
这样貌似解决了问题，但是又回到了方案2的问题，每次删除，前面都会有一个0，但是当你输入完成之后，因为进行了parseFloat操作，前导0又会消失，输入过程中不会消失，这样的用户体验很不好，按理说，每次触发了onChange事件，都应当执行parseFloat啊，可是这个处理后的值在value上没有体现，于是就有了下面的方案：在html处理一下：

```javascript
<div className="price">
    <span>{val || '0.0'}</span>
    <input value={val} placeholder="0.0" type="number" onChange={this.handleChange}/>
</div>
```
既然无法改变绑定在input上的值，但是这个值确实是去掉了前导0的，所以将input的透明度设置为0，显示的是span标签，这样的看似解决了问题，但是新的问题就是，在浏览器上看不到光标，看不到光标自然就没法移动光标，好在这个方案在IOS上测试，发现光标并不会因为因为input的透明度变化而变化，所以在手机上和在真实的input框里面输入数字体验差别不大，所以IOS上可以考虑这个方案；至于安卓，可以不使用type=number的方式；可以使用type=tel，因为安卓设置tel之后，唤起的数字键盘上有小数点，并且其他特殊字符也无法输入，只要在正则里限制小数点位数就好

```html
<input value={value} placeholder="0.0" type="tel" onChange={this.handleChange}/>
```
如果对小数点没有要求，IOS上可以设置模式，这样唤起的也只有数字键盘，并且没有小数点，也不可以切换输入法：

```html
<input value={value} placeholder="0.0" type="text" pattern="\d*" onChange={this.handleChange}/>
```
如果是浏览器的话，可以直接设置type=”text”，因为对唤起键盘没有要求，所以直接使用正则就可以控制输入。

### 终端判断
上面我们最终选择的方案就是，判断设备，然后渲染不同type的input，并且设置了不同的change方法，这样基本上能够满足需求，接下来就是判断终端了：

```javascript
cosnt isAndroid = /android/i.test(window.navigator.userAgent),
const isIOS = !!(window.navigator.userAgent.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/)),
```
这里还有一个问题，就是chrome，在正常打开的时候，这两个正则都为false，然而当切换为模拟器的时候，isIOS就会为true，在mac上打开的微信浏览器也是如此。
上述方案并不是很好，只能解决小需求，但是在价格位数不高的情况下，是可以正常使用的。然鹅我觉得要比自己去写一套假键盘一套假输入框，一套假光标要好一些，因为又会带来更多的新的问题，除非很有时间和精力去做这件事，否则不可取，此外，如果允许的话，h5内嵌与APP中，可以调native端同学的接口，效果肯定会比原生的好~

## 参考文档
1. [JS控制文本框只能输入数字和小数点](https://blog.csdn.net/itmyhome1990/article/details/77185691)
2. [input文本框输入时正则判断](https://www.haorooms.com/post/input_reg)
3. [js判断与过滤emoji表情的方法](https://blog.csdn.net/TKG09/article/details/53309455)
4. [移动端H5输入框、光标、数字键盘全假套件实现](https://zhuanlan.zhihu.com/p/30360629)