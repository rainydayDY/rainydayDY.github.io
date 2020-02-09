---
title: FormData--文件上传
date: 2018-4-14 13:53:30
tags:
- 文件传输
- form-data
categories: 前端
---
> 目标：理解文件上传时请求是以什么方式传输的

<p hidden><!--more--></p>
# 请求

### 文件上传
1. mutipart/form-data：将文件以二进制形式上传.
使用：设置form的enctype="mutipart/form-data"，其中enctype规定了表单发送到服务器的时候的编码方式，默认的编码方式是：application/x-www-form-urlencoded，不能传输文本和MP3等大型文件。
2. 当enctype为"mutipart/form-data"的时候，post请求的请求体如下：

```html
------WebKitFormBoundaryAaZg8fEDhzYGrPC7
Content-Disposition: form-data; name="file"; filename="5acc1cffdc4ce.jpg"
Content-Type: image/jpeg


------WebKitFormBoundaryAaZg8fEDhzYGrPC7--
```
3. 使用表单上传文件
```html
<form method="post" action="url" enctype="mutipart/form-data">
  <input type="file" name="file" />
  <button type="submit">上传</button>
</form>
```
4. 不用表单，使用FormData对象
> 通过FormData对象可以组装一组用 XMLHttpRequest发送请求的键/值对。它可以更灵活方便的发送表单数据，因为可以独立于表单使用。如果你把表单的编码类型设置为multipart/form-data ，则通过FormData传输的数据格式和表单通过submit() 方法传输的数据格式相同

```html
  <input type="file" @change="handleFile" />
```

```javascript
handleFile(e) {
    let file = e.target.files[0]
    const form = new FormData()
    form.append('file', file)
    request({
        url: '/student/uploadHeadImg',
        method: 'post',
        data: form
    }, response => {
        console.log(response)
    })
}
```

两种方式本质上没有区别，最终的结果都是体现在请求体中，请求体是一样的，都是使用字段分隔界线分割出分区，包含了描述头和主体部分，后端通过读取主体部分，筛出文件部分，在服务端将文件保存到磁盘。而传统的post请求，是通过键值对的方式传输的。文件是种复杂的数据格式，不能通过这种方式传输。
参考文章：
[HTTP 文件上传的基本原理](https://blog.csdn.net/aflyeaglenku/article/details/51644863)

[深入理解ajax系列第四篇——FormData](https://www.cnblogs.com/xiaohuochai/p/6539330.html)
