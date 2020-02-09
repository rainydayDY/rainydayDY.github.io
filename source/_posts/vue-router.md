---
title: 后台管理实现按钮级别的权限控制（小记）
date: 2018-5-25 13:53:30
tags:
- vue
- vue-router
- vuex
categories: 前端
---
> 需求：后台管理实现按钮级别的权限控制

<p hidden><!--more--></p>

解释：后台系统为公司内部人员使用，使用人的职位不同（即角色不同），则可以操作的权限不同，如超级管理员，可以定义不同角色，分配账号，分配权限菜单，高级运营可以审核信息，普通运营可以编辑文章等，不同的角色可以使用的菜单项不同，所以他们能够看到的菜单项也不一样，按钮级别的权限控制，顾名思义，就是要将权限做到按钮级别，就是不仅他们能够看到的菜单项不一样，可以使用的按钮也不一样，比如对用户的管理，涉及到“新增、编辑、删除、拉黑”，作为超级管理员，我可以操作任何一项，作为运营，我可以拉黑用户，但是作为客服，我只能看，剩下什么都做不了。（这些只是前端做的页面级的权限验证，后台也会根据token验证用户的权限的）（如下图所示）

![菜单示意图](http://wx2.sinaimg.cn/mw690/d83fac1bly1frm81k0xygj21610r0jtw.jpg)

描述：项目基于vue，使用了vuex+vue-router，模板为[Element](http://element-cn.eleme.io/#/zh-CN/component/installation)

所以上述的需求可以解释为如下：
> 不同的权限对应不同路由，同时侧边栏也需根据不同的权限，异步生成

思路：
1. 后台中的拥有最大权限的角色是超级管理员，所以他能够使用的菜单和按钮一定是最全的，也就是在需求确定的前提下，组件是确定的，即全部的菜单项是确定的，通常格式如下：
```javascript
export const asyncRouterMap = [
    {
        path: '/course',
        component: Layout,
        redirect: '/course/list',
        meta: {
            icon: 'table',
            id: 11
        },
        children: [
            {
                path: 'type',
                component: _import('course/type'),
                meta: { title: 'courseType', icon: 'course-type', id: 12 }
            },
            {
                path: 'child-type',
                component: _import('course/child-type'),
                meta: { title: 'childType', icon: 'child-type', id: 13 }
            },
            {
                path: 'list',
                component: _import('course/index'),
                meta: { title: 'courseList', icon: 'course-list', id: 14 }
            }
        ]
    }
    // id 事先不可知，在后台增加了菜单项之后才知道id，所有的菜单项是固定的，所以后期如果新增菜单项，这里改动也不会太大
];
```
2. 登录成功后，服务端返回token，根据这个token去拉取对应的菜单和按钮列表，后台返回的数据格式如下：
```javascript
{
    id: 11, name: "课程管理", buttonNames: null,children: [
      {
        id: 12, name: '课程类目' buttonNames: ['添加']
      }，
      {
        id: 13, name: '课程子类目' buttonNames: ['添加','编辑', '删除']
      }
    ]
}
```
3. 数据和路由都写好了，接下来就是做匹配了。用返回的菜单和路由做匹配，返回可以操作的路由，再通过router.addRoutes动态挂载路由，添加用户可以访问的路由，这里用到了导航守卫，我的理解是，存在管理员可能会在你操作数据，使用后台的同时，无论是有意还是无意的，可能去更改你的权限，为你增加或者减少权限，所以在每一次页面跳转都会去重新获取一遍菜单列表（这里为了提升用户体验，做了30分钟的缓存），并且在匹配菜单的时候，还会根据当前的地址，查询当前页面对应的操作按钮，之所以这样做，考虑到的是匹配路由也需要遍历菜单列表，如果到每个组件mounted的时候再去遍历菜单列表得到操作按钮，就相当于多做了一次遍历操作，而且这样造成的结果就会是页面有延迟，所以就合在了一起：
```javascript
// 在每次页面跳珠之前，在有token的前提下生成路由
router.beforeEach((to, from, next) => {
    if (getToken()) {
        if (to.path === '/login') {
            next({ path: '/' });
        } else {
            store.dispatch('GenerateRoutes', {
                isNow: false,
                path: to.path
            }).then(() => { // 根据roles权限生成可访问的路由表
                router.addRoutes(store.getters.addRouters); // 动态添加可访问路由表
            });
            next();
        }
    } else {
        next();
        next('/login'); // 否则全部重定向到登录页
    }
});
```
4. 使用vuex管理路由表，根据vuex中可访问的路由渲染侧边栏组件，根据vuex中生成的操作按钮渲染当前页面内的操作按钮：
```javascript
GenerateRoutes({ commit }, data) {
    // 这里自己封装了请求缓存函数
    networkRequest({
        url: '/permission/ht/getMenuList',
        method: 'get'
    }, response => {
        let accessedRouters;
        console.time('路由生成时间');
        accessedRouters = filterAsyncRouter(asyncRouterMap, response, data.path);
        // 侧边栏路由和当前页面操作按钮分离为两个状态
        commit('SET_ROUTERS', accessedRouters);
        commit('SET_BTNS', buttonNames);
        console.timeEnd('路由生成时间');
    }, {
        cacheName: 'menubar',
        cacheTime: 1800000,
        isNow: data.isNow
    });
}
```
5. 匹配路由，之所以我们能够对菜单列表和路由做匹配，是因为定义路由的时候配置了meta字段（[路由元信息](https://router.vuejs.org/zh/guide/advanced/meta.html)），通过对比菜单列表的id和路由的id，返回路由信息
```javascript
let buttonNames = [];
function hasPermission(menuArr, route) {
    if (route.meta && route.meta.id) {
        let childIndex = menuArr.findIndex(item => item.id === route.meta.id);
        if (childIndex !== -1) {
            // 渲染侧边栏的时候，也是通过设置meta 的title属性的，将菜单列表返回的菜单名字赋给title
            route.meta.title = menuArr[childIndex].name;
        }
        return childIndex !== -1;
    } else {
        return true;
    }
}
function filterAsyncRouter(asyncRouterMap, menuArr, path) {
    const accessedRouters = asyncRouterMap.filter(route => {
       //判断当前路由是否是用户可访问的路由
        if (hasPermission(menuArr, route)) {
            // 如果当前路由可以访问，再去匹配子路由
            if (route.children && route.children.length) {
                if (route.meta && route.meta.id) {
                    let childArr = [];
                    if (menuArr.find(item => item.id === route.meta.id)) {
                        let childIndex = menuArr.findIndex(item => item.id === route.meta.id);
                        route.meta.title = menuArr[childIndex].name;
                        childArr = menuArr[childIndex].children;
                        if (path) {
                            // 判断当前的路由和当前页面的一级路由是否一致,如果一致,查询二级路由下的操作按钮
                            let pathArr = path.split('/');
                            if (route.path === `/${pathArr[1]}`) {
                                let index = route.children.findIndex(item => item.path === pathArr[2]);
                                buttonNames = childArr.find(item => item.id === route.children[index].meta.id).buttonNames;
                            }
                        }
                    }
                    route.children = filterAsyncRouter(route.children, childArr);
                } else {
                    return false;
                }
            }
            return true;
        }
        return false;
    });
    return accessedRouters;
}
```

6. 生成侧边栏,使用的是Element的NavMenu组件。遍历生成的路由即可，菜单的名字使用的是meta的title。

7. 生成按钮，用computed计算得到vuex中的按钮名称数组，实际上页面内已经写好所有的按钮和绑定函数了，这里直接用v-if进行渲染。
```html
<el-button  @click="handleCreate"  v-if="buttonNames.includes('添加')">添加</el-button>
```
```javascript
computed: {
    buttonNames() {
        return this.$store.state.permission.btnArr;
    }
}
```

此后台是在[vue-element-admin](https://github.com/PanJiaChen/vue-element-admin)的基础上更改的，我认为他做的就非常好，而且读懂他的代码后，再去完成自己的需求，会变得容易很多，还有详细的参考文档，我是从去年12月份开始使用这个模板的，从这项目学到了很多东西，也是这个项目给我指明了一些学习方向，勾起了我的探索兴趣，非常感谢。
从这里你可以学到的：
1. 更好的理解登录权限控制
2. 更好的编码习惯和思考习惯

### 参考文档

1. [手摸手，带你用vue撸后台 系列二(登录权限篇)](https://juejin.im/post/591aa14f570c35006961acac#heading-10)

### 分享阅读清单

1. [head部分的清单](https://github.com/Amery2010/HEAD#%E6%9C%80%E5%B0%8F%E6%8E%A8%E8%8D%90)

#### meta 标签
```html
<meta charset="utf-8">
<meta http-equiv="x-ua-compatible" content="ie=edge"><!-- † -->
<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
<!-- 以上3个meta标签必须放在head之前，以确保文档的正确呈现 -->
<!-- 强制 IE 8/9/10 使用其最新的渲染引擎 -->
<meta http-equiv="x-ua-compatible" content="ie=edge">
<!-- 360浏览器选择渲染引擎 -->
<meta name="renderer" content="webkit|ie-comp|ie-stand">
<!-- QQ浏览器在指定方向上锁定屏幕（锁定横/竖屏） -->
<meta name="x5-orientation" content="landscape/portrait">
<!-- QQ浏览器全屏显示此页面 -->
<meta name="x5-fullscreen" content="true">
<!-- QQ浏览器页面将以“应用模式”显示（全屏等）-->
<meta name="x5-page-mode" content="app">
<!-- UC浏览器 -->
<!-- 在指定方向上锁定屏幕（锁定横/竖屏） -->
<meta name="screen-orientation" content="landscape/portrait">
<!-- 全屏显示此页面 -->
<meta name="full-screen" content="yes">
<!-- 即使在“文本模式”下，UC 浏览器也会显示图片 -->
<meta name="imagemode" content="force">
<!-- 页面将以“应用模式”显示（全屏、禁止手势等） -->
<meta name="browsermode" content="application">
<!-- 在此页面禁用 UC 浏览器的“夜间模式” -->
<meta name="nightmode" content="disable">
<!-- 简化页面，减少数据传输 -->
<meta name="layoutmode" content="fitscreen">
<!-- 禁用的 UC 浏览器的功能，“当此页面中有较多文本时缩放字体” -->
<meta name="wap-font-scale" content="no">
```

2. [解锁多种JavaScript数组去重姿势](https://juejin.im/post/5b0284ac51882542ad774c45)

```javascript
// 执行效率最高的两种数组去重的方法,arr为待去重的数组
const tmp = new Map();
return arr.filter(item => {
    if (!tmp.has(item) && tmp.set(item, 1));
})
// 或者使用set
let newArr = [...new Set(arr)]
```
