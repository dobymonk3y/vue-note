# 页面骨架搭建
## 移动端的meta设置
```css
    <meta charset="utf-8">
    <meta name="viewport"
          content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no">
```

## 增加的依赖项
babel的使用参考 http://www.ruanyifeng.com/blog/2016/01/babel.html

下面的polyfill和babel-runtime 的安装 和 `.babelrc`中的'transform-runtime' 我看文档好像有这个插件就不用再自己转了，会自己去做转换的。 -------------------？这个有待验证

### babel-polyfill插件
Babel默认只转换新的`JavaScript句法（syntax）`，而不转换新的API，比如`Iterator`、`Generator`、`Set`、`Maps`、`Proxy`、`Reflect`、`Symbol`、`Promise`等全局对象，以及一些定义在全局对象上的方法（比如`Object.assign`）都不会转码。
举例来说，ES6在`Array`对象上新增了`Array.from`方法。Babel就不会转码这个方法。如果想让这个方法运行，必须使用`babel-polyfill`，为当前环境提供一个垫片。详细清单可以查看 `babel-plugin-transform-runtime`模块的[definitions.js](https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js) 文件

为了完整使用 ES6 的 API ，我们只能安装这个插件：

```bash
$ npm install -save-dev babel-polyfill
```

使用方法，在入口文件最上层引入
```javascript
import 'babel-polyfill'  // es6 到 es5 的补丁转换
```

### babel-runtime
`babel-polyfill` 与 `babel-runtime` 是两个概念

`babel-polyfill` 是对浏览器缺失API的支持。比如浏览器可能没有`Array.from()` 方法。

`babel-runtime` 是为了减少重复代码而生的。 babel生成的代码，可能会用到一些_`extend()`， `classCallCheck()` 之类的工具函数，默认情况下，这些工具函数的代码会包含在编译后的文件中。如果存在多个文件，那每个文件都有可能含有一份重复的代码。
`babel-runtime`插件能够将这些工具函数的代码转换成`require`语句，指向为对`babel-runtime`的引用，如 `require('babel-runtime/helpers/classCallCheck')`. 这样， `classCallCheck`的代码就不需要在每个文件中都存在了。
当然，最终你需要利用`webpack`之类的打包工具，将`runtime`代码打包到目标文件中。
```bash
"babel-runtime": "^6.0.0"
```

### FastClick
处理移动端 click 事件 300 毫秒延迟， 由 FT Labs 开发，Github 项目地址：https://github.com/ftlabs/fastclick 。
```bash
"fastclick": "^1.0.6"
```
使用如下：在main.js中加入如下代码
```javascript
import fastclick from 'fastclick'
fastclick.attach(document.body)  // 解决移动端点击延迟300毫秒

在main.js中加入是因为该项目是spa单页应用，这里设置在body上，body下的所有点击事件都被处理了。
```

## header 组件的开发
先根据页面结构，把最顶端的header组件开发出来

## tab 导航切换组件的开发
切换不同的tab，把下面对应的内容跳转到具体的页面，该导航就会存在直接输入地址不会对应的激活，但是这个要看场景，该项目是移动端的音乐app，不会以网页形式直接访问，应该是包裹在一个app里面的，就是说不能直接访问地址的形式

