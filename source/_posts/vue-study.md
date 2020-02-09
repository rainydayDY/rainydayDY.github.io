---
title: vue 学习
date: 2017-08-30 15:12:07
tags:
- todolist
- 实例
categories: "前端"
---
### 前言 ###
&emsp;&emsp;最近忙的没时间写博客，写了标题却没时间来写内容，又到月底了，总结时，发觉这一个月好像没学会什么东西，做了博客，重新搭建了服务器，还做了一点移动端适配的东西，其他的都是杂七杂八的小秘书工作。学习落下很多，必须要补回来。

> 想要改变现在的处境，只有一条出路。
<p hidden><!--more--></p>

* vue基础知识介绍

&emsp;&emsp;MVVM框架；与后台交互使用vue-resource这个插件，类似于jquery的ajax功能，和后台进行交互，不仅可以实现get，同时可实现post，还有jsonp，使用此插件，不需要依赖其他第三方的插件，就可以实现基本的业务需求，Vue特点：易用、高效、灵活。
#### 灵活 ####
>* 声明式渲染，基于模型视图
* 组件系统(提取可复用的公共部分)
* 客户端路由
* 大规模状态管理
* 构建工具

#### 高效 ####
>* min+gzip的运行大小是16kb
* 超快虚拟DOM（1.0版本没有虚拟DOM，2.0中出现虚拟的DOM，最后一次更改会被绑定到最后的DOM中）
* 优化省心

#### 指令介绍 ####
1、v-model&emsp;&emsp;在表单中使用<br>
2、v-text&emsp;&emsp;文本的渲染<br>
3、v-show&emsp;&emsp;控制DOM的显示和隐藏<br>
4、v-if&emsp;&emsp;控制DOM的存在与否<br>
5、v-bind&emsp;&emsp;绑定属性<br>
6、v-for&emsp;&emsp;循环数组<br>
7、v-on&emsp;&emsp;事件绑定<br>
8、过滤器（filter）&emsp;&emsp;只显示某些字段，对接口返回的字段进行业务转换
9、组件&emsp;&emsp;创建vue实例，引入vue.js和vue-resource.js；<br>
<pre>
var vm=new Vue({
    el:"app", /*模型的监听范围*/
    data:{},
    mounted:function(){

    }, /*实例化完成后执行*/
    methods:{}
    })
    Vue.filter();
</pre>
不需要依赖其他第三方框架。
使用的时候引入vue.js和vue-resource.js即可，在自己的javacript里面写逻辑，创建实例。

-----分割线------
#### vue实现待办事项列表todolist ###

题目要求：

1. 单个事项的添加 新添加的项目为未完成状态
2. 单个事项的删除 点击可以直接删除，不需要弹出确定框
3. 单个事项的完成状态的修改 未完成到已完成，已完成到未完成都可以
4. 已完成事项的显示开关 开关打开时显示所有事项，开关关闭时只显示未完成事项，在开关关闭状态下，当某事项由未完成变成已完成， 需要隐藏该事项
5. 所有已完成事项的清除 点击可以直接删除，不需要弹出确定框，在隐藏已完成事项的状态下也可以清除

分析：

&emsp;&emsp;所有的事件存储应该是数组形式，每一个事件为对象，对象包含的属性中事件内容是必须的，完成状态和显隐状态可以是后期添加的，当用户改变事件的状态时，再去添加这个完成状态的属性，如果当前事件的这个属性存在，直接取反即可，显示和隐藏所有已完成事件，很明显，不是针对某一个事件进行的，所以需要遍历所有事件，对标记为已完成状态的事件添加显示隐藏状态。删除和添加就是对数组的操作，push和splice就够用了，删除所有已完成事件需要稍微动一下脑筋，因为是删除多个元素，在删除过程中，如果是从前往后删除，每删除一个，后面的元素下标都会发生变化，所以倒序删除是个好的解决方法。

实现：
<img src="http://s4.sinaimg.cn/orignal/003XwVVpzy7fcS5iLoD33&690" style="width:100%;height:auto">
```javascript
var vm = new Vue({
    el: '#app',
    data: {
        listarray:[{
            content:'十里河下车行不行'
        },{
            content:'绿萝要记得浇水哦，垃圾记得到哦'
        }],
        message:'',
        pbtn:true
    },
    methods:{
        addlist:function(){
            var obj={content:this.message}
            this.listarray.push(obj);
            this.message='';
        },
        done:function(things){
            if(typeof(things.ischeck)=='undefined'){
                Vue.set(things,'ischeck',true)
            }else {
                things.ischeck=!things.ischeck;
            }
        },
        deletelist: function(i){
            // 删一个数组元素
            this.listarray.splice(i,1);
        },
        toogleshow:function(){
          var _this=this;
          //当pbtn为true时，点击效果为隐藏所有已完成的
          this.listarray.forEach(function(list,index){
              if(list.ischeck){
                if(typeof(list.showall)=='undefined'){
                  Vue.set(list,'showall',_this.pbtn)
                }else {
                  list.showall=_this.pbtn;
                }
              }
          })
          this.pbtn=!this.pbtn;
        },
        deleteall:function(){
          var _this=this;
          var i=this.listarray.length;
          while (i--) {
            if(this.listarray[i].ischeck){
              console.log(i+'对应数组'+this.listarray[i]);
              this.listarray.splice(i,1);
            }
          }
        }
    }
})

```
实现效果：
<img src="http://s3.sinaimg.cn/orignal/003XwVVpzy7fcSG6yRka2&690" style="margin:auto">
