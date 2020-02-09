
---
title: vue-cli构建项目
date: 2018-3-12 13:53:30
tags:
- vue
- element-ui
- sass
categories: 前端
---
> 目标：掌握使用vue-cli构建项目，引入element-ui、sass
<p hidden><!--more--></p>


### 全局安装 vue-cli
$ npm install --global vue-cli
### 创建一个基于 webpack 模板的新项目
$ vue init webpack 'project-name'
### 安装依赖
$ cd 'project-name'
$ npm install

## 项目介绍
我司的项目是基于vue的，项目构建无要求，目的只有一个方便维护和开发，所以我是根据项目原型自己和自己需求来构建的。
基本上的技术组成是：ES6+sass+element-ui，目前是这样的，后期可能会扩展。
参考的是一个开源的项目：[vue-element-admin](https://github.com/PanJiaChen/vue-element-admin"后台管理模板")

项目目录（一个清晰的项目目录可以快速找到所需，并进行开发）：
1. components 可以共享的组件放在这里
2. utils 可以共享的函数.js文件放在这里
3. pages 真正的页面放在这里
4. api 登录等请求的接口函数放在这里
5. assets 存放不能使用icon的静态资源
6. icons/svg 存放所有用到的icon，UI需要提供**svg**格式，使用时当做字体来处理
7. styles/element.scss 存放覆盖element-ui原有样式的文件
8. styles/index.scss 存放样式变量还有一些公共样式

***
## 首先引入element-ui
参考文章：[elment-ui](http://element.eleme.io/#/zh-CN/component/installation)
#### 安装：
npm install element-ui -D
#### 使用
修改入口文件：main.js
```javascript
import Vue from 'vue'
import Element from 'element-ui'
Vue.use(Element, {
    size: 'medium'
})
// 其中size属性是控制所有组件的大小的
```
由于项目中一些组件的功能和element-ui的组件功能一样，只是样式不一样，所以新建了一个**element.scss**，用来覆盖原有样式。
```css
.el-pager li.active{
  background-color: #4bbbfa !important;
}
.el-pagination.is-background .el-pager li{
  color:#999;
}
.el-pager li{
  border-radius: 4px !important;
  width:30px;
  height: 30px;
}
```
## 项目中使用sass
#### 安装
npm install node-sass --save-dev
npm install sass-loader --save-dev
#### 配置 webpack.base.con.js
```javascript
{
    test: /\.scss$/,
    loaders: ["style", "css", "sass"]
}
```
#### 使用sass
```css
<style lang="scss" scoped>
/* 引入全局的css */
@import '../styles/index.scss';
/* 每个使用到全局样式的地方都需要这样写 */
```
## 处理icon
参考文章：[手摸手，带你优雅的使用icon](https://juejin.im/post/59bb864b5188257e7a427c09#heading-6)

项目中使用的所有icon(不限制单色OR多色)都来自于iconfont.cn或者由UI提供，格式为svg
#### 安装依赖
npm install svg-sprite-loader -D
目的：打包svg为svg-sprite，类似于我们最初使用的css-sprite
#### 配置webpack
vue-cli默认情况下会使用url-loader对svg进行处理，所以直接引入svg-sprite-loder会有冲突，解决方案如下：
```javascript
{
  test: /\.svg$/,
  loader: 'svg-sprite-loader',
  include: [resolve('src/icons')],
  options: {
    symbolId: 'icon-[name]'
  }
},
{
  test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
  loader: 'url-loader',
  exclude: [resolve('src/icons')],
  options: {
    limit: 10000,
    name: utils.assetsPath('img/[name].[hash:7].[ext]')
  }
```
### 创建icon组件
```html
<template>
  <svg :class="svgClass" aria-hidden="true">
    <use :xlink:href="iconName"></use>
  </svg>
</template>

<script>
export default {
    name: 'svg-icon',
    props: {
        iconClass: {
            type: String,
            required: true
        },
        className: {
            type: String
        }
    },
    computed: {
        iconName() {
            return `#icon-${this.iconClass}`
        },
        svgClass() {
            if (this.className) {
                return 'svg-icon ' + this.className
            } else {
                return 'svg-icon'
            }
        }
    }
}
</script>

<style scoped>
.svg-icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}
</style>

```
#### 导入icon
创建一个存放icon的文件夹，如src/icon，在此文件夹下创建一个index.js，代码如下：
```javascript
import Vue from 'vue'
import SvgIcon from '@/components/svg-icon.vue'// svg组件

// register globally
Vue.component('svg-icon', SvgIcon)

const requireAll = requireContext => requireContext.keys().map(requireContext)
const req = require.context('./svg', false, /\.svg$/)
const iconMap = requireAll(req)

```
#### 使用icon

```html
<svg icon-class="使用icon的名字"/>
如：<svg icon-class="star"/>
```
## 处理请求 axios
简介：基于promise的HTTP请求客户端。
