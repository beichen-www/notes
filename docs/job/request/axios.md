## axios 封装

#### 安装
#### npm：**npm install axios**

#### yarn：**yarn add axios**

#### 封装函数

| 配置    | 说明         |
| ------- | ------------ |
| baseURL | Api 配置     |
| timeout | 设置超时时间 |
| headers | 设置请求头 |


#### 添加 request.js 文件

```javascript
import axios from 'axios'
import { Message } from 'element-ui'
import store from '@/store'
import { getToken } from '@/utils/auth'

const service = axios.create({
  baseURL: process.env.VUE_APP_BASE_API,
  timeout: 60000
})

service.interceptors.request.use(
  config => {
    if (store.getters.token) {
      let headers = {
        'Accept': 'application/json',
        'Authorization': 'Bearer ' + getToken(),
      };
      config.headers = headers
    }
    return config
  },
  error => {
    console.log(error)
    return Promise.reject(error)
  }
)

service.interceptors.response.use(
  response => {
    if (response.status !== 200) {
      Message({
        message: response.statusText || 'Error',
        type: 'error',
        duration: 5 * 1000
      })
      return Promise.reject(new Error(response.statusText || 'Error'))
    } else {
      return response.data
    }
  },
  error => {
    if (error && error.response) {
      switch (error.response.status) {
        case 241:
          error.message = '缺少参数,请检查参数';
          Message.error({ message: '缺少参数,请检查参数' });
          break;
        case 400:
          error.message = '错误请求';
          break;
        case 401:
          Message.error({ message: '您的权限不足,请联系管理员!' });
          error.message = '未授权，请重新登录';
          redirect('/login')
          break;
        case 403:
          Message.error({ message: '您的权限不足!' });
          error.message = '拒绝访问';
          break;
        case 404:
          Message.error({ message: '未找到该资源' });
          error.message = '未找到该资源';
          break;
        case 405:
          Message.error({ message: '请检查您的请求方法是否匹配' });
          error.message = '请求方法未允许';
          break;
        case 408:
          Message.error({ message: '请求超时' });
          error.message = '请求超时';
          break;
        case 500:
          Message.error({ message: '服务器出错了' });
          error.message = '服务器端出错';
          break;
        case 501:
          error.message = '网络未实现';
          break;
        case 502:
          Message.error({ message: '网络错误' });
          error.message = '网络错误';
          break;
        case 503:
          error.message = '服务不可用';
          break;
        case 504:
          error.message = '网络超时';
          break;
        case 505:
          error.message = 'http版本不支持该请求';
          break;
        default:
          error.message = `未知错误${error.response.status}`;
      }
    } else {
      error.message = "连接到服务器失败";
    }
    return Promise.reject(error)
  }
)

export default service

```

###  使用
```javascript

import request from '@/utils/request'

export function login(data) {
  return request({
    url: '/login',
    method: 'post',
    data
  })
}
```

#### 相关链接

[官网地址](https://axios-http.com/zh)
