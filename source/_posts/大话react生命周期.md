---
title: 大话react生命周期
date: 2018-12-21 15:14:26
tags:
- 场景
- use
- redux
categories: 前端
---
> 需求： 分析生命周期的触发时机 && react中在生命周期的触发中常见业务场景
<p hidden><!--more--></p>

组件的生命周期，是指一个组件从创建到销毁的过程，有一个比喻做的很形象，组件的生命周期可以看做是一只蚂蚁爬过一根吊绳，这只蚂蚁从头爬到尾，会依次触动不同的卡片挂钩，这个卡片挂钩就是生命周期中的钩子函数。而事实上如果蚂蚁的行动路线不同，有的卡片就不会碰到，有些卡片一定会碰到。一定会碰到的卡片对应于生命周期中必经的钩子函数，如compomentDidMount, componentWillUnmount,render，而不会碰到的可能是componentWillReceiveProps这样的钩子。

## 要点概述
1. 路由参数的变化，页面如何刷新？
2. 如何缓存组件？在哪个阶段对组件进行缓存？
3. 登录的时候，在哪里进行token判断？
4. render函数在什么时候触发？
5. 详情页传来的参数在什么时候存，localstrage在什么时候写入和读取？
6. 全局事件注册时机？（点击空白处关闭弹窗）
7. scrollTop滚动时机？

## 生命周期阶段概述
1. constuctor --- 初始化阶段，构造函数，初始化state
2. componentWillMount --- 组件即将被渲染，此时DOM还未形成，只会调用一次
3. render --- 返回jsx
4. componentDidMount --- 组件挂载时执行，此时已形成DOM
5. componentWillReceiveProps --- 父组件的传递的props发生改变或者父组件re-render的时候，
6. componentWillUpdate --- 父组件的传递的props发生改变，先触发此钩子函数
7. componentDidUpdate --- render执行完之后执行，DOM确实已更新
8. componentWillUnmount --- 组件卸载，销毁

> Tip: 挂载——组件被实例化并且被挂载到dom树上的过程称为挂载
>      卸载——组件被移除Dom树

## 父子组件的生命周期
初次加载执行顺序：

```javascript
父组件constructor
父组件WillMount
父组件render
子组件constructor
子组件WillMount
子组件render
子组件DidMount
父组件DidMount
父组件WillUnmount
子组件WillUnmount
```
当子组件内的表单项改变了，此时的生命周期变化为：

```javascript
父组件WillUpdate
父组件render
子组件WillReceiveProps
子组件WillUpdate
子组件render
子组件DidUpdate
父组件DidUpdate
```
## 实例 && 场景

### 路由参数的变化，刷新页面
实际开发中，常常有这样的场景，列表对应一个组件，一个路由，详情对应一个组件，一个路由，页面可能会因为参数的不同，需要来回切换，比如说一个tab页，有三种状态：待查看、已查看、已完成，实际上布局一样，请求的地址也一样，只是参数不同，切换的时候需要重新发送请求，这种场景就会有两种方案：
***方案1***
监听切换tab的事件，发送请求
***方案2***
监听路由参数的变化，发送请求
方案1，直接在事件上做处理，但是如果是详情页不同，从一个详情页跳转到另一个详情页，路由没有发生变化，只是参数发生了变化，这时候页面不会刷新，请求也不会发送，组件也没有发生变化，此时会触发的生命周期函数只有两个componentWillReceiveProps和componentDidUpdate，所以可以像下面这样处理：
```javascript
    componentWillReceiveProps(nextProps) {
        // 仅当两个路由完全相同，参数不同的页面切换时，才会触发。
        if (nextProps.location.search !== this.props.location.search) {
            this.getDetail(nextProps.location.search);
        }
    }
```
### 缓存页面数据
react中没有类似vue中keep-alive那样的缓存组件的功能，如果想要缓存组件数据，需要自己手动去做存储，选择存在redux中是比较好的选择，那么在什么时机存储呢？路由切换，上一个组件就会销毁，然后新的组件的生命周期会开始，所以对应于不同路由间切换，存储数据可以选择在组件销毁阶段或者组件挂载阶段，选择其一即可：
```javascript
    componentWillUnmount() {
        // 在用户未自己进行保存的前提下，才去存redux
        if (!this.props.isSave) {
            this.handleSaveRedux();
        }
    }
```
想象一下，如果是在两个详情页之间切换路由，怎么进行缓存呢？上面已经提到要在componentWillReceiveProps这个生命周期中处理，我们需要缓存上一个组件的数据，同时要获取新组件的数据：
```javascript
    componentWillReceiveProps(nextProps) {
        if (nextProps.location.search !== this.props.location.search) {
            const { location: { search, pathname } } = this.props;
            this.handleSaveRedux(`${pathname}${search}`); // 先存上一个路由的数据
            this.getDetail(nextProps.location.search); // 后取下一个组件的数据
        }
    }
```
如果详情页由父子组件组成，子组件做表单处理，父组件做数据处理，发送请求，提交请求等，那么保存表单数据就会是子组件处理，这时候，如果想要利用上面的钩子函数，需要注意父子组件的钩子函数的执行顺序，一定是先在子组件内保存数据，然后在父组件中拉取新页面的数据，但是此时父组件的componentWillReceiveProps优先于子组件执行，那么上面的可以改成：
```javascript
    componentWillReceiveProps(nextProps) {
        // 父组件
        if (nextProps.location.search !== this.props.location.search) {
            // 优先内层FormPage保存数据，保存完一个页面的数据，此页面带入新页面的数据
            setTimeout(() => {
                this.getDetail(nextProps.location.search); // 后取下一个组件的数据
            }, 0);
        }
    }
    componentWillReceiveProps(nextProps) {
        // 子组件
        if (nextProps.location.search !== this.props.location.search) {
            const { location: { search, pathname } } = this.props;
            this.handleSaveRedux(`${pathname}${search}`); // 先存上一个路由的数据
        }
    }
```
### 判断登录态的时机
如果页面处于登录态，正常跳转，如果没有，那么就要跳转到登录页，这个登录态的判断，通常以token形式存在，单页面应用中，为了避免刷新页面导致、redux数据消失，所以存在localstorage中， 每次进入页面前都先判断一遍，token是否存在，如果这样的判断在每个页面都写一遍肯定是不行的，所以可以将这样的判断写在basicLayout组件里，担任所有其他页面的父组件的角色。如果登录全由我们自己控制，这个登录条件的判断写在componentDidMount或者componentWillMount里面都可以，因为layout里面的判断逻辑只会执行一遍，如果是加了微信的授权登录，中途有个跳转到第三方页面的过程，所以这时候一定要注意生命周期，未登录的状态下，保证先获取到token，再去判断token，
```javascript
// 登录组件
    componentDidMount() {
        if (!code) {
            getThirdCode(); // 微信授权登录，获取code值，获取完成之后又跳回登录页，此时携带code，有了code值之后不再刷新页面
        }
    }
// layout组件
    componentDidMount() {
        if (!token) {
            History.push('/login'); // 子组件的componentDidMount先执行，所以这时再去跳转，不会刷洗页面，组件没有销毁。
        }
    }
```
先执行子组件的componentDidMount，再执行父组件的componentDidMount，好处是，跳转一次三方页面获取成功code值之后，虽然父组件又判断了一遍，但是此时路由没有变化，没有再次执行componentDidMount，所以没有再次去获取code值，code值页存在state中，没有变，但是如果此时改成下面：
```javascript
// layout组件
componentWillMount() {
    if (!token) {
        History.push('/login');
    }
}
```
此时的问题就是父组件先判断没有token，跳到登录页，登录页发现没有code跳到三方页，三方页又跳回登录页，url中有code,layout组件中的componentWillMount先于登录组件的componentDidMount执行，执行了一次跳转，此时跳回登录页无code值，就又会跳回三方页获取code，所以会不断的循环，登录页不断的刷新，用户永远都进入不了登录页进行登录。。

### render函数
调用时机：
1. 组件初始化
2. 组件的props和state发生改变
3. 组件内调用forceUpdate函数
4. 父组件调用了render函数，即使子组件的props和之前一样，也会调用render

> render函数并不做实际的渲染动作，它只是返回一个jsx描述的结构，最终由React来操作渲染过程

### 参数缓存时机
场景是这样的，用户进入详情页的方式有很多种，可以从列表页进入，也可以通过某个链接直接访问，当用户通过点击链接进入详情页的时候，发送请求时需要携带链接地址中的参数，如果直接点击进来不做token校验，没有问题，直接进入然后获取参数即可，如果需要做token校验，并且中间会使用第三方登录（意味着中间会刷新页面），所以这个参数的存储就成了问题，存redux里是不行的，因为中间有刷新过程，所以考虑存到localstorage里，并且在layout中做处理，此时一定是先存，后取，所以父组件的这个钩子函数的执行要早于子组件的钩子函数，子组件的请求通常写在componentDidMount中，那么父组件的选择只能是componentWillMount或者constuctor中。
```javascript
    componentWillMount() {
        const { location: { search } } = this.props;
        const params = getParams(search, 'id');  
        params && writeLocalStorage('id', params);
    }
```
### 点击空白处关闭弹窗
绑定DOM事件，一定要在DOM 形成之后，即componentDidMount阶段：
```javascript
    componentDidMount() {
        document.addEventListener('click', this.closeMenu);
    }
```
此外，点击弹框内的button要正确执行，就不得不阻止事件冒泡，react中的阻止事件冒泡：
```javascript
    closeTag = (e) => {
        e.nativeEvent.stopImmediatePropagation();
        e.preventDefault && e.preventDefault();
    }
```
### scrollTop滚动时机
场景描述：一个详情页，可以添加多个卡片，每次添加完一个卡片，滚动条都要滚动到最底端，以便用户可以直接点击加号进行卡片添加和编辑刚刚新增加的卡片。
从实现上讲，添加卡片，其实就是渲染一个卡片列表，实质上是往数组里面push一个新的对象，即：
```javascript
    handleAdd = () => {
        this.setState(({ cardList }) => ({
            cardList: cardList.concat(newItem),
        }));
        this.scrollToBottom();
    }

     scrollToBottom = () => {
        const content = this.listPage; // 通过设置ref获取 <div ref={listPage => this.listPage = listPage}></div>
        content.scrollTop = content.scrollHeight - window.document.body.clientHeight;
    }
```
基本上思路是这样的，然后尝试添加卡片，发现滚动没有效果啊，？？？？
理一下思路，首先要想到的是setState是异步行为，可是在添加卡片的过程中，都触发了哪些生命周期呢？
首先肯定是state变化了，所以顺序应该是这样的：
```javascript
    componentWillUpdate()  //state已经改变
    render() // 新的卡片添加到虚拟dom中
    componentDidUpdate() // 此时添加卡片的操作才算完成，新的卡片才添加到真正的DOM结构中
```
所以我们能想到，要延后一个进程去执行滚动事件，在componentDidUpdate阶段获取到的scrollHeight才是添加了新卡片dom的滚动条高度，所以上面的滚动方法可以改成下面这样：
```javascript
    scrollToBottom = () => {
        const content = this.listPage;
        setTimeout(() => {
            content.scrollTop = content.scrollHeight - window.document.body.clientHeight;
        }, 0);
    }
    // 或者在直接将scrollToBottom延后一个进程，修改handleAdd事件
    setTimeout(this.scrollToBottom, 0);
```
> Tip：滚动的元素，一定是子元素的高度高于父元素的高度，并且父元素设置了overflow:auto or scroll，此时对父元素添加scrollTop才生效

```javascript
    setTimeout(() => {
        this.printStr(1);
    }, 0);
    setTimeout(this.printStr(2), 0);
    setTimeout(this.printStr, 0);
    printStr = (n) => {
        console.log(n);
    }
    // 执行结果： 2 1 undifined
```

## 总结
以上的几个场景基本上就是最近一个月的踩坑史，因为对生命周期的理解欠缺，导致出了很多的bug，遂作此篇。出bug并不丢人，写代码是一种修行。