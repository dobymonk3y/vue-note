# 页面骨架搭建
## 移动端的meta设置
```css
    <meta charset="utf-8">
    <meta name="viewport"
          content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no">
```

## 增加的依赖项
babel的使用参考 http://www.ruanyifeng.com/blog/2016/01/babel.html

### babel-polyfill插件
> 本说明参考 [该页](http://www.imooc.com/article/11157) 
Babel默认只转换新的`JavaScript句法（syntax）`，而不转换新的API，比如`Iterator`、`Generator`、`Set`、`Maps`、`Proxy`、`Reflect`、`Symbol`、`Promise`等全局对象，以及一些定义在全局对象上的方法（比如`Object.assign`）都不会转码。
举例来说，ES6在`Array`对象上新增了`Array.from`方法。Babel就不会转码这个方法。如果想让这个方法运行，必须使用`babel-polyfill`，为当前环境提供一个垫片。详细清单可以查看 `babel-plugin-transform-runtime`模块的[definitions.js](https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js) 文件

为了完整使用 ES6 的 API ，我们只能安装这个插件：

```bash
$ npm install -save-dev babel-polyfill
```

### babel-runtime
适合库和框架，针对使用到的新API 会提出打包到 build 文件中。
```bash
"babel-runtime": "^6.0.0"
```

### FastClick
处理移动端 click 事件 300 毫秒延迟， 由 FT Labs 开发，Github 项目地址：https://github.com/ftlabs/fastclick 。
```bash
"fastclick": "^1.0.6"
```
