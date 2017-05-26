# Element-ui 饿了么UI
## 使用ElementUI

官方文档：http://element.eleme.io/#/zh-CN/component/installation

全局引入：
```javascript
-- main.js --
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-default/index.css'
Vue.use(ElementUI)
```

## Element主题定制

http://element.eleme.io/#/zh-CN/component/custom-theme

### 安装
全局安装工具
```
npm i element-theme -g
```
安装依赖默认主题
```
npm i element-theme-default -D
```
### 初始化变量文件
在项目根目录下执行
```
et -i ./src/comm/style/element-ui/element-variables.css
```
生成变量文件后，可以修改里面的变量、

### 编译主题
保存文件后，到命令行里执行 et 编译主题，如果你想启用 watch 模式，实时编译主题，增加 -w 参数；如果你在初始化时指定了自定义变量文件，则需要增加 -c 参数，并带上你的变量文件名
```
et -c ./src/comm/style/element-ui/element-variables.css -o ./src/comm/style/element-ui/theme

> ✔ build theme font
> ✔ build element theme
```

# 使用中遇到的坑和技巧

## NavMenu 导航菜单动态渲染 
在文档中没有找到怎么只给数据然后渲染出菜单来。经过摸索后，稍微封装了下。
思路如下：
1. 观察SubMenu，Menu-Item，Menu-Group 使用方式和所需要的数据，然后定制数据结构
2. 通过分解组件加 v-if 进行判断组装

代码如下：
ElMenuItemEx.vue
```
<template>
  <el-menu-item v-if="menu.type == 'item' && menu.route " :index="menu.index" :route="menu.route">
    <template v-if="menu.route.type == 'inner'">
      <i :class="menu.icon"></i>{{ menu.lable }}
    </template>
    <template v-if="menu.route.type == 'out'">
      <i :class="menu.icon"></i><a :href="menu.route.path" target="_blank">{{ menu.lable }}</a>
    </template>
  </el-menu-item>
</template>

<script>
  export default {
    name: 'el-menu-item-ex',
    props: {
      menu: {
        type: Object
      }
    },
    data () {
      return {}
    }
  }
</script>
<style lang="less" rel="stylesheet/less">

</style>

```

