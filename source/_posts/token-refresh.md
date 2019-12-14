---
title: 关于token在刚刚过期后的刷新问题
date: 2019-11-30 22:29:23
tags:
  - javascript
---

### Token无缝刷新

前后端分离的系统，假如这个系统是给客服人员使用，客服人员可能要在长达八个小时的时间上不停地工作；为了安全考虑，不给token设定永久的时常，而是给它一个可刷新时间区域：*在当前token刚刚过期后的一段时间发起的请求，系统可以无缝刷新一次token，从而为使用者带来更好的体验*，这是非常重要的。

#### Axios中的实现

##### 原理简单阐述

实现很简单，在拦截器当中检测返回数据，检测到刚刚过期可以刷新的情况，则在拦截器中直接发起刷新token的请求，获得到新token后保存，再执行一次失败的请求，执行完毕返回。

##### 会遇到的问题

axios是异步执行的，假如在刷新token的过程中又有其他携带旧token并收到需要刷新的标识后，又如何处理呢？

设置一个全局标识，表明当前是否正在刷新token，如果正在刷新，就拒绝。这样显然是不好的。在一次需求中可能要发起多个请求这是很正常的事情。为了不让其他请求失败，我的做法是，创建一个异步任务顺序执行器，在刷新token期间还携带旧token并返回过期的请求，则重新将本请求加入到这个执行器的末尾；而执行器则是在首次检测到过期时进行初始化，并开始执行第一个任务，也就是刷新token任务。

##### 代码

放代码之前我先给出一些解释：我并没有使用常规的http状态码来判断响应结果，而是在后端程序中讲所有非系统错误的大部分异常全部封装为了自定义的错误码并返回给前台，我在拦截器中自己来判断返回结果。

``` javascript
/* eslint-disable no-unused-vars */
'use strict'

import Vue from 'vue'
import axios from 'axios'
import router from '../router.js'
import { Notification, Message } from 'element-ui'

// Full config:  https://github.com/axios/axios#request-config
// axios.defaults.baseURL = process.env.baseURL || process.env.apiUrl || ''
// axios.defaults.headers.common['Authorization'] = AUTH_TOKEN
// axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded'

let config = {
  // baseURL: process.env.baseURL || process.env.apiUrl || ''
  // timeout: 60 * 1000, // Timeout
  // withCredentials: true, // Check cross-site Access-Control,
}

const _axios = axios.create(config)

_axios.interceptors.request.use(
  function (config) {
    // Do something before request is sent
    if (!config.url.includes('http')) {
      config.url = 'http://127.0.0.1:4399' + config.url
    }
    const token = localStorage.getItem('token')
    if (token) {
      config.headers.Authorization = token
    }
    return config
  },
  function (error) {
    // Do something with request error
    return Promise.reject(error)
  }
)

let refreshTokenFlag = false
let requestList = []

function executeRequests(index) {
  if (index < requestList.length) {
    requestList[index]().then(() => {
      executeRequests(index + 1)
    })
  } else {
    refreshTokenFlag = false
    requestList.splice(0, requestList.length)
  }
}

// Add a response interceptor
_axios.interceptors.response.use(
  function (response) {
    console.log(` >>> 拦截器开始`)
    console.log(response.config.url)
    console.log(response.config.headers.Authorization)
    console.log(response.data)
    // Do something with response data
    /** 
     * 在这里返回Promise.reject会进入代码的catch块;
     * 返回Promise.resolve或正常数据会进入then块;
     * 不返回 = 返回undefined也会进入then块
     */
    // 成功
    if (response.data.code === 1) {
      return response.data
    }
    if (response.data.code === 2 || response.data.code === 4) {
      // 失败 || 无权访问
      Message({
        message: response.data.data || '网络繁忙！',
        type: 'error',
        center: true
      })
      return Promise.reject(response.data)
    } else if (response.data.code === 3) {
      // 需要登录验证权限
      const redirectUrl = router.currentRoute.fullPath
      router.push({
        path: '/login',
        query: {
          redirectUrl
        }
      })
      return Promise.reject(response.data)
    } else if (response.data.code === 5) {
      // 需要刷新token
      const userId = response.data.data
      if (refreshTokenFlag) {
        // 正在刷新
        return new Promise((resolve, reject) => {
          requestList.push(async function () {
            try {
              const r = await _axios(response.config)
              resolve(r)
            } catch (err) {
              reject(err)
            }
          })
        })
      } else {
        // 未刷新
        refreshTokenFlag = true
        return new Promise((resolve, reject) => {
          requestList.push(async function () {
            try {
              const result = await _axios.post(`/sign/refreshToken/${userId}`)
              localStorage.setItem('token', result.data)
              const r = await _axios(response.config)
              resolve(r)
            } catch (err) {
              reject(err)
            }
          })
          executeRequests(0)
        })
      }
    }
  },
  function (error) {
    console.log('Ajax系统错误')
    Notification.error({
      title: '系统错误',
      message: `${error || 'No message available'}`
    })
    return Promise.reject(error)
  }
)

Plugin.install = function (Vue, options) {
  Vue.axios = _axios
  window.axios = _axios
  Object.defineProperties(Vue.prototype, {
    axios: {
      get() {
        return _axios
      }
    },
    $axios: {
      get() {
        return _axios
      }
    },
  })
}

Vue.use(Plugin)

export default Plugin

```

##### 其他

其实无痛刷新token的方法有很多，我在之前用到的方法则是在springmvc的Interceptor以及ResponseBodyAdvice中进行token的刷新，在返回结果中携带一个token刷新标志以及刷新后的token；这带来了一个问题：所有的token刷新都会在后端来解决，对于一个小系统，小网站来说，很可能会影响程序执行效率（当然大系统应该有自己的一套方案，这我就不知道了，若是有大佬路过也很期待您能在评论中给出一个小建议，感激不尽）。

在实现通过前端来刷新的这个方法后，经过测试发现，体验还不错，在刷新的过程会有比较低的延迟但完全可以接受（延迟若是很大的话可能本身列入队列的请求也很复杂），所以写下这篇文章以便记录。

