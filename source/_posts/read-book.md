---
title: 算法 学习笔记
date: 2018-6-25 13:53:30
tags:
- 基础算法
categories: 前端
---
> 目标：学习算法的思想，逐步提升自己

<p hidden><!--more--></p>

## 开闭原则
定义：Software entities should be open for extension,but closed for modification.
翻译：软件实体对扩展应当是开放的，对修改应当是关闭的。即扩展时不修改或者尽量少修改原有代码，以保证软件系统的稳定性。
实现：
1. 识别出系统的可变化和可能可变化的点（做之前先思考）
2. 抽象化可变化的点【接口】，并单独实现可变化的点的抽象【接口的实现】
3. 系统依赖抽象化【接口】而不是依赖于实现。
4. 对实现的路由单独封装，最好以配置的方式提供。（配置不一定是配置文件，可以是代码，但形式一定要简单直观）
5. 需要改变功能的时候，只需要提供新的满足抽象化【接口】的实现，并且更改下路由的配置即可
参考文档：
[前端和开闭原则](https://github.com/cnsnake11/blog/blob/master/%E5%85%B6%E5%AE%83/%E5%89%8D%E7%AB%AF%E5%92%8C%E5%BC%80%E9%97%AD%E5%8E%9F%E5%88%99.md?from=timeline&isappinstalled=0)

## 抽象思维

### 费波那契函数
费波那契问题是，如果它们按照如下规则，兔子的数量是如何从一代到一代的增长：

1. 每对发育成熟的兔子每月产下一对新的后代；
2. 兔子在出生后的第二个月发育成熟；
3. 老兔子不死；
如果在1月引入一对未发育成熟的兔子，那么在年终时将会有多少对兔子？
如果这道题加上，未成熟兔子第二月成熟，第三月生产，才是斐波那契数列。。。
```javascript
let Fib = n => {
    if (n < 2) {
        return n;
    }else {
        return Fib(n-1) + Fib(n-2);
    }
};
```

### 回文

“回文串”是一个正读和反读都一样的字符串，比如“level”或者“noon”等等就是回文串。

```javascript
let checkPalindromic = str =>  {
    let len = str.length;
    if (len <= 1) {
        return true;
    }else {
        return str[0] === str[len-1] && checkPalindromic(str.substring(1,len-1));
    }
};
```

### 二分查找

折半查找，适用于顺序排列的数组，通过改变索引的值，来减少查找范围
```javascript
let halfSelect = (arr,aim,min,max) => {
    let mid = parseInt((min + max) /2);
    if (aim === arr[mid]) {
        return mid;
    }else if (aim < arr[mid]) {
        return halfSelect(arr,aim,min,mid-1);
    }else if (aim > arr[mid]) {
        return halfSelect(arr,aim,mid+1,max);
    }
};
```

### 汉诺塔

```javascript
let hanoi = (n,a,b,c) => {
    if (n === 1) {
        console.log(`move ${n} from ${a} to ${c}`);
    }else {
        hanoi(n-1,a,c,b);
        console.log(`move ${n} from ${a} to ${c}`);
        hanoi(n-1,b,a,c);
    }
};
```
