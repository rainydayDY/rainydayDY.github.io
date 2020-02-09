---
title: js 深浅拷贝
date: 2018-12-22 14:22:45
tags:
- redux
- object
categories: 前端
---
> 目标：理解数据的基本类型和引用类型 && 数据的深浅拷贝和赋值
<p hidden><!--more--></p>

## 基本数据类型 && 引用数据类型
基本数据类型（原始类型）：number string undifined null boolean
基本数据类型的比较，是值得比较，只要值相等，默认他们就是相等的比如：
```javascript
    let a = 'dy';
    let b = 'dy';
    console.log(a === b); // true
```
引用数据类型（复合数据类型）： Object Array func date...
引用类型是存放在堆内存中的，变量实际上是一个存放在栈内存中的指针，这个指针指向堆内存中的地址。
引用类型的比较是引用的比较，是否指向同一块内存地址：
```javascript
    let a = {
        name: 'dy'
    };
    let b = {
        name: 'dy'
    };
    console.log(a === b); // false
```
对于赋值，基本数据类型赋的是真正的值，即传值，而引用类型赋的是地址，即传址：
```javascript
    let a = 'dy';
    let b = a;
    b = 'wd';
    console.log(a); // dy
    console.log(b); // wd
    let c = {
        name: 'dy',
        hobbies: ['coding', 'reading'],
    };
    let d = c;
    d.name = 'wd';
    console.log(d.name); // wd
    console.log(c.name); // wd
    console.log(c === d); // true
```
## 浅拷贝
继续上面的例子
```javascript
    let newObj = shallowCopy(c);
    newObj.name = 'dh';
    newObj.hobbies = ['playing'];
    console.log(c); 
    /* c = {
     *     name: 'wd',
     *     hobbies: ['playing'],
     * };
     */
```
和赋值不同的是，浅拷贝新开辟了内存，不过只拷贝了一层，如果对象里面还包含子对象，那么子对象的引用还是相同的，我们通常使用的
```javascript
    // 对象的浅拷贝
    let newObj = Object.assign({}, obj);
    let newObj = {
        ...obj,
    };
    // 数组的浅拷贝
    let newArr = arr.concat();
    let newArr = [...arr];
    let newArr = arr.slice();
    // 通用浅拷贝
    function shallowCopy(originData) {
        if (typeof originData !== 'object') return;
        let newData = oirgindata instanceof Array ? [] : {};
        for (let key in origindata) {
            if (originData.hasOwnProperty(key)) {
                newData[key] = originData[key];
            }
        }
        return newData;
    }
```
以上两者都属于浅拷贝；如果我们明确知道里面不包含子对象，可以使用，或者包含子对象，但是子对象我们也进行了浅拷贝，上面的方法是可以使用的。与浅拷贝对应的就是深拷贝了，深拷贝应该是对对象的子对象以及后代的所有子对象的值都进行赋值，并且重新开辟新的内存。

 ## redux中存储的数据并不是只有action可以改
 理想情况下，我们使用redux一方面是为了在单页面中共享状态，另一方面是为了清楚数据流向。
 redux中createStore方法，用于创建一个保存程序状态的store。改变store中数据的唯一方法是调用store的`dispatch()`方法。
 可是对于store中的数据嵌套层级比较深，类似于下面：
 ```javascript
    const state = [{
        id: 1,
        name: 'dy',
        todayList: [{
            price: 0,
            hobbies: [{
                type: '1',
                name: 'study',
                price: 100,
            }, {
                type: '2',
                name: 'eat',
                price: 500,
            }]
        }]
    }]
 ```
 在页面初始化的时候，你对hobbies里面的数据做了改变：
 ```javascript
    componentDidMount() {
        const currentState = state[0];
        const { todayList: { hobbies }  } = currentState;
        hobbies.forEach((item) => {
            item.price /= item.price;
        });
    }
 ```
 此时你打开redux，查看里面的数据，执行完这一步，数据一定是在没有dispatch的情况下变了的。其实也不能算是redux的锅，是我们自己在使用过程中改变了原始数据，第一步，赋值，值里面的内容是个对象，所以currentState和store中的第一条数据还是指向同一块内存的，引用是相同的，后面又用了forEach循环，一定会改变原始数组的，所以就出现了后面的结果，没有触发dispatch，但是数据也发生了改变。
 上面的问题的解决方案有两种，其一为在取数据的时候就直接使用深拷贝，以后随意更改都无所谓，其二是，取出数据之后我们队数组操作的时候，不使用forEach这种会改变原数组的函数，使用map。以下列举会改变原数组的函数
 ```javascript
    splice();
    forEach();
    push();
    pop();
    unshift();
 ```
 ## 深拷贝
 深拷贝的实现可以递归调用浅拷贝的方法
 ```javascript
    function deepCopy(originData) {
         if (typeof originData !== 'object') return;
        let newData = oirgindata instanceof Array ? [] : {};
        for (let key in origindata) {
            if (originData.hasOwnProperty(key)) {
                newData[key] = typeof originData[key] === 'object' ? deepCopy(origindata[key]) : originData[key];
            }
        }
    }
 ```
 递归会带来性能问题，所以用的时候要慎重。
 还可以使用lodash的deepCopy方法。


 ## 参考文章
 1. [js 深拷贝 vs 浅拷贝](https://juejin.im/post/59ac1c4ef265da248e75892b)
 2. [JavaScript专题之深浅拷贝](https://juejin.im/post/59658504f265da6c415f3324)
