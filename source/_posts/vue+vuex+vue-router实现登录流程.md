---
title: vue实现登录流程
date: 2018-3-18 13:53:30
tags:
- vue
- vuex
- vue-router
categories: 前端
---
> 目标：弄清楚登录流程，以及vuex和vue-router的使用。着重看路由的导航守卫

<p hidden><!--more--></p>
> 需求描述：用户通过手机号获取验证码的方式注册或者登录，进入主页，侧边栏有用户头像和昵称，点击头像可以进行信息的修改，修改提交后侧边栏要求在不刷新页面的前提下也同步改变；用户刷新浏览器后或者关闭浏览器不必进行重新登录（一段时间内），如果用户清理缓存，应立刻跳转到登录页重新登录，此时无论从哪个入口进入都会重定向到登录页。

实现思路：由于项目基于vue，属于单页面应用，多组件通信，考虑用到vuex，然而vuex中存储的状态在刷新页面后会重置为初始状态，所以用户的登录态要在本地进行存储，我用到的是cookie，页面跳转通过判断这个登录态是否存在，而决定是跳到即将跳转到的页面还是跳回登录页，所以需要注册一个全局前值守卫rouer.beforeEach(),根据to.path决定next.path。

暂时没有API，所以数据都是自己模拟的。
### 模拟数据

```javascript
const userInfo = [
{
    'id': '456',
    'account': '13002208106',
    'nickname': null,
    'avtor': null,
    'qq': null,
    'address': null
}
]
export default {
  // 登录，返回用户账号
  login(phone) {
      return userInfo.find(item => item.account === phone).account
  },
  // 获取用户信息
  getUserInfo(token) {
      return userInfo.find(item => item.account === token)
  }
}
```
### 登录
提交表单，触发登录的action
```javascript
this.$refs.form.validate(valid => {
    if (valid) {
        this.$store.dispatch('LoginByTel', this.form).then(() => {
            this.$router.push({
                name: 'my-class'
            })
        }).catch(() => {
            this.$message.error('登录失败')
        })
    }
})
```
vuex中action处理，然后提交改变state
```javascript
// import netWorkRequest from '@/utils/netWorkRequest'
import Cookies from 'js-cookie'
import constant from '../constant'
// 请求的数据文件

const state = {
    token: Cookies.get('TokenKey'),
    name: null,
    avtor: null,
    userInfo: null
}

const mutations = {
    SET_TOKEN: (state, token) => {
        state.token = token
    },
    SET_NAME: (state, name) => {
        state.name = name
    },
    SET_AVATAR: (state, avtor) => {
        state.avtor = avtor
    },
    SET_INFO: (state, info) => {
        state.userInfo = info
    }
}
const actions = {
    LoginByTel({
        commit
    }, userInfo) {
        const response = constant.login(userInfo.tel)
        commit('SET_TOKEN', response)
        Cookies.set('TokenKey', response)
    },

    // 获取用户信息
    getUserInfo({
        commit,
        state
    }) {
        const response = constant.getUserInfo(state.token)
        commit('SET_INFO', response)
        commit('SET_NAME', response.nickname ? response.nickname : response.account)
        commit('SET_AVATAR', response.avtor ? response.avtor : 'https://gitee.com/uploads/58/1650058_rainyday66.png?1511432604')
    }
}

export default {
    state,
    actions,
    mutations
}

```
### 拦截路由，导航守卫部分
```javascript
import router from './router'
import store from './store'
import Cookies from 'js-cookie'

router.beforeEach((to, from, next) => {
    if (Cookies.get('TokenKey')) {
        if (to.path === '/login') {
            // 存在token情况下，再去进入登录页则自动跳回首页
            next({
                path: '/'
            })
        } else {
            if (!store.state.user.userInfo) { // 判断当前用户是否已拉取完user_info信息
                store.dispatch('getUserInfo')
            }
            next()
        }
    } else {
        next()
        next('/login')
    }
})

```
