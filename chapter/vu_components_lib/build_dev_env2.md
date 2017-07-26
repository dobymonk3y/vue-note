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
