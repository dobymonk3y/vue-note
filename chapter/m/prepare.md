# 准备工作

## 使用stylus编写css
先安装，记得安装loader和原本的stylus库
```bash
npm i stylus-loader stylus --save
```
安装完成之后的vue模版中的写法如下
```css
<style scoped lang="stylus" rel="stylesheet/stylus">
</style>
```

接下来介绍一点常用的规范css
### variable.styl
变量，整体风格颜色，字体大小设置
### reset.styl
重置风格，业内有统一的css。
### base.styl
基础的css
### icon.styl
图标css
### index.styl
入口引用 
```css
@import "./reset.styl"
@import "./base.styl"
@import "./icon.styl"
```
### mixin.styl
stylus 函数，在使用到的地方引入


基础的css文件准备好后，在main.js中引入该css
```javascript
import 'common/stylus/index.styl'
```