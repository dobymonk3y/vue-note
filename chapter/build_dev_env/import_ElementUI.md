# 准备工作
配置一些laoder

## 引入ElementUI

为了节约时间。使用ElementUI - Vue2 来布局和使用他的一些样式组件

官方文档：http://element.eleme.io/#/zh-CN/component/installation

全局引入：
```javascript
-- main.js --
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-default/index.css'
Vue.use(ElementUI)
``` 
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

