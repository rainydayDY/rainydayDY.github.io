---
title: post提交数据的四种编码方式
date: 2018-10-24 16:38:57
tags:
- axios
- https
categories: 前端
---
> 需求描述：解决使用axios库以post方式提交请求遇到的问题。
<p hidden><!--more--></p>

最近的项目中，有好多同学在发送post请求时出现了问题，对照swagger文档，请求方式和传参都没错，而且swagger上也返回正常结果，使用postman也没有问题，但是在开发时，却总是返回500，和后端同学联调，后端同学表示接收不到参数，因为现在的开发中普遍使用的都是json，而且不管是前端同学还是后端同学都是使用框架开发，框架大多数都内置了自动解析常见数据格式的功能，所以好多同学对编码方式都不太了解，在此做出总结：
## application/x-www-form-urlencoded
最常见的post编码方式，浏览器的原生form表单，默认就是以这种编码方式提交，大部分服务器端语言都对这种方式有很好的支持，请求体格式如下：

我们使用的请求库axios在使用post方法提交数据时，默认为application/json格式，后端也大多接收此种编码方式的数据，所以我们通常按照这种方式发送请求：
```javascript
axios.post(url, { 
    key: value,
}).then((res) => {}).catch(e => reject(e));
```
在实际开发中，post的字段非常少的情况下，比如只有一个id，一些后端同学就不用json的方式解析，我们再以此方式发送请求，就会报错，axios官方给出了几种解决办法：
```javascript
const params = new URLSearchParams();
params.append('param1', 'value1');
params.append('param2', 'value2');
axios.post('/foo', params);
```
这种方式，并不是所有浏览器都支持，所以在项目中，我们使用下面这种方式：
```javascript
const querystring = require('querystring');
axios.post('http://something.com/', querystring.stringify({ foo: 'bar' }));
```
这种方式提交的数据是按照key1=value1&key2=value2的方式编码，key和value都进行了URI转码，所以我们可以设置编码方式，然后对key和value进行URI转码，实现如下：（设置transformRequest允许在向服务器发送请求前，修改数据）

```javascript
api.post(url, {
    key: value
}, {
    transformRequest: [function (data) {
        let ret = '';
        for (let it in data) {
            if (data.hasOwnProperty(it)) {
                    ret += encodeURIComponent(it) + '=' + encodeURIComponent(data[it]) + '&';
            }
        }
        return ret;
    }],
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    withCredentials: false,
}).then((res) => {}).catch(e => reject(e));
```
目前我们的项目中大多采用第二种方式和第三种方式。

## application/form-data
在使用表单上传文件时，需要以此方式发送post请求，如果是使用原生表单，需要设置enctype为’multipart/form-data’，如果不使用表单，可以说使用FormData对象，它可以独立于表单使用，在我们封装图片上传组件的时候可以采用这种方式，这两种方法传输的数据格式相同，本质上没有区别，请求体都是使用分隔界线分区，具体表现形式如下：

## application/json
这种编码方式我们再实际开发中使用的最多，请求主体是序列化的JSON字符串，表现形式如下：



## text/xml
暂未用到。

具体使用哪种方式，后端同学会在swagger中标出，比如第一种方式，后端在parameters里面会标出“（query）”，而第三种方式，标识是”（body）”
