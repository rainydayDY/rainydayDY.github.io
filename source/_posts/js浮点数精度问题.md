---
title: js浮点数精度问题
date: 2018-12-10 16:38:57
tags:
- 浮点数
- 加减乘除 && show
categories: 前端
---
> 内容描述：解决浮点数运算出现不准确的问题和多位小数显示问题
<p hidden><!--more--></p>

在日常开发中，最常出现浮点数运算的情况就是价格的计算，通常价格的存储单位为分，但是显示单位为元，所以在展示、提交、请求的过程中，难免要进行价格单位的转换（即浮点数的乘除），如果算个合计，还要进行多位数相加，如果有优惠减免之类的，还会涉及到加减混合，然而，并不是所有的数的运算结果都会如我们所愿，比如0.1+0.2就不会得0.3，33.3/100也不会得0.333，诸如此类，是什么导致这种问题呢？
JavaScript 中所有数字包括整数和小数都只有一种类型 — Number。它的实现遵循 IEEE 754 标准，使用 64 位固定长度来表示，也就是标准的 double 双精度浮点数（相关的还有float 32位单精度）。
0.1 转成二进制表示为 0.0001100110011001100(1100循环)，所以我们在进行计算的时候需要先将其转换为整数，对整数进行运算就不会有问题，然后对运算结果再除以当时转换为整数时乘的倍数，问题就解决了，所以，基本上所有的运算都依赖于乘法运算，乘法运算中最关键的又是指数运算(**)。

### 实现加减乘除
> 1.1 * 100 = ?

###  乘法
```javascript
function multiplyFloat(...args) {
    if(!args) return 0;
    if (args.length < 2) return args[0];
    let m = 0;
    const items = args.map((item) => {
        let newItem = String(item);
        newItem.split('.')[1] && (m += newItem.split('.')[1].length);
        return newItem.replace('.', '');
    });
    return items.reduce((prev, curr) => (prev * curr)) / 10 ** m;      
}
```
###  加法
> 0.1 + 0.2 = ?

```javascript
function addFloat(...args) {
   if(!args) return 0;
   if (args.length < 2) return args[0];
   const decimals = args.map((item) => {
       let newItem = String(item);
        return newItem.split('.')[1] ? newItem.split('.')[1].length : 0;
   });
   cosnt baseNum = 10 ** Math.max(...decimals);
   return args.reduce((prev, curr) => multiplyFloat(prev, baseNum) + multiplyFloat(curr, baseNum)) / baseNum;
}
```
### 减法
```javascript
function subtractionFloat(...args) {
    if(!args) return 0;
    if (args.length < 2) return args[0];
    const decimals = args.map((item) => {
        let newItem = String(item);
         return newItem.split('.')[1] ? newItem.split('.')[1].length : 0;
    });
    cosnt baseNum = 10 ** Math.max(...decimals);
    return args.reduce((prev, curr) => multiplyFloat(prev, baseNum) - multiplyFloat(curr, baseNum)) / baseNum;
}
```
### 除法
> 33.3 / 100 = ?

```javascript
function divideFloat(...args) {
    if(!args) return 0;
    if (args.length < 2) return args[0];
    return args.reduce((prev, curr) => {
        const p = `${prev}`;
        const c = `${curr}`;
        let r1 = p.split('.')[1] ? p.split('.')[1].length : 0;
        let r2 = c.split('.')[1] ? c.split('.')[1].length : 0;
        const m = p.replace('.', '') / c.replace('.', '');
        return multiplyFloat(m, 10 ** (r2-r1));
    });
}
```
### 展示浮点数
> 1.4000000000000001 正确展示

```javascript
function displayFloat(val, precision = 12) {
    // toPrecision() 方法可在对象的值超出指定位数时将其转换为指数计数法。precision范围为[1, 21]
    return +parseFloat(Number(val).toPrecision(precision));
}
```
### 参考文章
1. [JavaScript 浮点数陷阱及解法](https://github.com/camsong/blog/issues/9)