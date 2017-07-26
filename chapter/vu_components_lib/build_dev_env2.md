# 环境搭建-组件打包环境

组件打包环境 目前我的理解是，把要独立出来的组件 全部放到了一个固定的目录，和 开发测试环境分开了。

组件打包的目录如下：
```bash
| - rootxxx                 # 项目根目录（项目名）
    |- src                  # 用于组件开发,vue-li默认生成的目录
        |- components       # 我们所有的组件都放在这个里面 
          |- hello          # hello组件目录
            |-Hello.vue     
            |-index.js      # 模块化暴露文件
```

本次只配置组件打包和使用，其他的需求以后再慢慢深入了解

## 编写第一个组件
**src/components/hello/Hello.vue**
```javascript
<template>
  <div class="hello">
    <h1>{{ title }}</h1>
    <h3>{{ msg }}</h3>
    <ul>
      <li><a href="http://router.vuejs.org/" target="_blank">vue-router</a></li>
      <li><a href="http://vuex.vuejs.org/" target="_blank">vuex</a></li>
      <li><a href="http://vue-loader.vuejs.org/" target="_blank">vue-loader</a></li>
      <li><a href="https://github.com/vuejs/awesome-vue" target="_blank">awesome-vue</a></li>
    </ul>
  </div>
</template>

<script>
  export default {
    props: {
      title: String
    },
    data () {
      return {
        msg: 'hello word'
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
    background-color: #9ebdd7;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }
</style>

```
**src/components/hello/index.js**
```javascript
import Hello from './Hello'
export default Hello
```

到此为止我们的组件写完了，测试怎么测试呢？

* 直接和平时开发webapp一样，可以直接引入该vue文件。
* 还可以使用 Vue.use(xx) 安装的方式来引入文件，这种方式下面讲

## 实现Vue.use全局安装我们的组件
由于我们写的是组件库。所以要暴露很多不同的组件。需要一个统一的js来进行安装。
**src/index.js**
```javascript
/**
 *
 * <pre>
 *  Version         Date            Author          Description
 * ---------------------------------------------------------------------------------------
 *  1.0.0           2017/07/25     zhuqiang        -
 * </pre>
 * @author zhuqiang
 * @version 1.0.0 2017/7/26 13:54
 * @date 2017/7/26 13:54
 * @since 1.0.0
 */
// 这个难道是一个es6的补丁？正常的vue-cli中已经包含了补丁了应该。所以这个暂时不知道去掉后怎么重现故障
import 'core-js/fn/array/find-index'

import Hello from './components/hello'
let VueComponentsLib = {
  Hello
}

const install = function (Vue, opts = {}) {
  Object.keys(VueComponentsLib).forEach((key) => {
    Vue.component(key, VueComponentsLib[key])
  })
}

// auto install
if (typeof window !== 'undefined' && window.Vue) {
  install(window.Vue)
}
// 把 install方法添加到 VueComponentsLib 中
export default Object.assign(VueComponentsLib, {install})
// module.exports = Object.assign(tlzVueElUi, {install})   // eslint-disable-line no-undef

```

这个测试方法如下：
在examples/main.js中添加如下代码
```javascript
import VueComponentsLib from '../src/index'
Vue.use(VueComponentsLib)
```
好了，这个被全局安装了，至于按需安装的话，我就没有研究了，后台系统全局安装能满足我的需求了。
