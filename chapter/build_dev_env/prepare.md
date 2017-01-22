# 准备工作
配置一些laoder

## 引入ElementUI

为了节约时间。使用ElementUI - Vue2 来布局和使用他的一些样式组件
[ElementUI使用](../element_ui.md)


## 引入less
使用less来编写css样式
官方:
* https://github.com/webpack/less-loader
* http://lesscss.cn/


1. 安装：
```bash
npm install --save-dev less-loader less
```
需要注意上面的命令，一次性安装了2个，单独安装less-loader。在运行时会报错的。

2. 使用
在.vue style 中声明; 不然不会起作用
```javascript
<style lang="less">
```

## 开始写主页布局
..... 省略

## 左侧菜单路由切换


## 配置 axios 请求资源
https://github.com/mzabriskie/axios
安装
```bash
npm install axios --dev
```
使用示例：
```javascript
 import Axios from 'axios';
  Axios.get('/api/test/list')
      .then(function (response) {
        console.log(response);
      })
      .catch(function (error) {
        console.log(error);
      });
```

## 配置开发环境的跨域代理
https://github.com/chimurai/http-proxy-middleware

```javascript
--- config/index.js ---
    proxyTable: {
      '/api': {
        target: 'http://192.168.6.70:8080',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    }
    pathRewrite： 这里配置的是 正则表达式，以/api 开头的将会被用与‘’替换掉。也就是说
    程序中你写访问：/api/task/xxxx. 被处理之后：http://192.168.6.70:8080/task/xxxx
```


