---
title: js实现文件下载
date: 2019-06-11 20:35:02
tags:
- Blob
- http header
categories: 前端
---
> 内容描述：js对后端返回的流文件进行下载处理
<p hidden><!--more--></p>

## 需求背景
需求是这样的，你需要在某个网站上填写一份申请表，然后再提交，系统自动审核申请表是否合格，所以系统需要给你提供一个申请表的模板，你下载下来然后去填写，系统生成的模板文件，可能会根据一些信息做调整，所以这个文件不是固定存在的。

## 实现

上面已经提到了，文件是根据查询条件不同生成的，所以没办法通过点击链接的形式进行下载，你需要去请求服务端，服务端给你返回流文件，实现的方式有多种，但是首先，你要理清下面几种概念：

1. content-disposition
2. form提交和ajax提交的Accept

以上概念都是基于响应头和请求头的，所以你要熟悉http请求

> content-disposition：在常规的HTTP应答中，Content-Disposition 消息头指示回复的内容该以何种形式展示，是以内联的形式（即网页或者页面的一部分），还是以附件的形式下载并保存到本地。

content-disposition控制用户请求所得的内容存为一个文件的时候提供一个默认的文件名，文件直接在浏览器上显示或者在访问时弹出文件下载对话框。attachment即以附件的形式下载。
这个属性所有浏览器都支持，由服务端同学设置在response header里，设置完之后，是这样的

```
content-disposition: attachment;filename=%E5%95%86%E5%93%81%E6%98%8E%E7%BB%86.xlsx
```
理解了这个属性，你就可以发送请求给服务端，问题又来了，我们发送请求的方式也影响了这个属性的有效性。

### form方式提交，服务端设置content-dispoistion

实现文件下载，你可以创建一个隐藏的表单，将method设置为正确的请求方式，提交表单，服务端接受到请求后，返回给我们一个二进制流，因为表单是可以接受这种类型的响应值的，所以这个文件就会按照响应头设置的那样，将服务端返回的流文件存为一个文件，并按照设置好的文件名，进行下载或者打开操作。

```javascript
export function downloadFile(apiPath, params, isFullUrl) {
    let f = document.createElement('form');
    f.style = 'display:none';
    f.action = isFullUrl ? apiPath : globalConfig.apiHost + apiPath;
    f.method = 'POST';
    for (let key in params) {
        if (params.hasOwnProperty(key)) {
            let i = document.createElement('input');
            i.type = 'hidden';
            i.value = params[key] != undefined ? params[key] : null;
            i.name = key;
            f.appendChild(i);
        }
    }
    document.body.appendChild(f);
    f.submit();
    document.body.removeChild(f);
}
```

```javascript
// 表单提交时的request header是这样的
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Content-Type: application/x-www-form-urlencoded
```
### ajax提交，提取流文件下载

使用ajax提交，你会看到返回值是一堆标红的乱码值，此刻也没有触发自动下载，是因为使用ajax请求，返回值的类型只能是json、text/html、xml类型，不能够接受流类型，但是仍然可以获得文件的内容，该文件内容将保留在内存中，JavaScript无法和磁盘进行交互，否则这会是一个严重的安全问题，它也无法调用到浏览器的下载处理机制和程序，所以直接使用ajax触发请求是不能够实现文件下载的。

```javascript
// 使用ajax请求时的requst header
Accept: application/json, text/plain, */*
Content-Type: application/json;charset=UTF-8
```
所以我们的方法是，可以接受这个流文件，创建一个blob对象，一个具有download属性的a标签，自动触发a标签的点击，实现blob对象的下载。
我们需要对返回的内容做处理，以axios为例：

```javascript
    // 首先设置响应类型
    api.post(url, { ...params }, {
            responseType: 'blob', // 二进制流
    }).then((res) => {
        const url = window.URL.createObjectURL(new Blob([res.data]));
        let link = document.createElement('a');
        link.style.display = 'none';
        link.href = url;
        link.setAttribute('download', 'filename');
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        window.URL.revokeObjectURL(url); // 释放掉blob对象
    }   
```
> URL.createObjectURL() 静态方法会创建一个 DOMString，其中包含一个表示参数中给出的对象的URL。这个 URL 的生命周期和创建它的窗口中的 document 绑定。这个新的URL 对象表示指定的 File 对象或 Blob 对象。这个url是唯一的，指向的是创建的blob对象。

但是上面的实现方式会有兼容性问题，IE就不支持这样的下载方式，所以需要对IE单独处理
```javascript
    if (window.navigator.msSaveOrOpenBlob) {
        navigator.msSaveBlob(blob, 'filename');
    }
```
微软的开发文档中是这样描述的：
当一个站点调用这个方法，表现和服务端设置响应头部Content-Disposition是完全一样的。并且这个方法是异步的且没有回调函数的。

### 使用思考
这两种实现方式各有优缺点，使用的时候还是考虑场景，比如说我要记录下载时长，添加loading状态，我就需要使用后者，因为我可以拿到返回值，确定返回的时间，而使用form的方式，我们不知道请求是什么时候返回的，因为返回了就会自动下载。此外，对于参数的格式，form不能使用json的方式提交，也是一种限制。但是前者兼容性好。

## 完整的代码

```javascript
/**
  * 下载文件
  * @param    {string}  url     下载地址
  * @param    {object}  params  下载所需参数
  * @param    {string}  name    下载的文件名
  * */
export function downloadFile(url, params, name) {
    return new Promise((resolve, reject) => {
        axios.post(url, { ...params }, {
            responseType: 'blob', // 二进制流
        }).then((res) => {
            if (res) {
                const blob = new Blob([res.data]);
                const currentTime = new Date();
                const filename = `${name}_${currentTime.getHours()}时${currentTime.getMinutes()}分.xlsx`;
                if (window.navigator.msSaveOrOpenBlob) {
                    navigator.msSaveBlob(blob, filename);
                } else {
                    let url = window.URL.createObjectURL(blob);
                    let link = document.createElement('a');
                    link.style.display = 'none';
                    link.href = url;
                    link.setAttribute('download', filename);
                    document.body.appendChild(link);
                    link.click();
                    document.body.removeChild(link);
                    window.URL.revokeObjectURL(url); // 释放掉blob对象
                }
                resolve();
            } else {
                reject('下载失败');
            }
        }).catch((err) => {
            reject(err);
        });
    });
}
```

## 参考文章
1. [JavaScript 中 Blob 对象](https://juejin.im/entry/5937c98eac502e0068cf31ae)
2. [web页面实现文件下载的几种方法](https://www.cnblogs.com/voiphudong/p/3284724.html)
3. [msSaveOrOpenBlob method](https://msdn.microsoft.com/en-us/library/hh772332(v=vs.85).aspx)
4. [Content-Disposition](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Disposition)

