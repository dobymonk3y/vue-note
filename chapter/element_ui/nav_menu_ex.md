# NavMenu 导航菜单动态渲染 
在文档中没有找到怎么只给数据然后渲染出菜单来。经过摸索后，稍微封装了下。
思路如下：
1. 观察SubMenu，Menu-Item，Menu-Group 使用方式和所需要的数据，然后定制数据结构
2. 通过分解组件加 v-if 进行判断组装

代码如下：
ElMenuItemEx.vue
```javascript
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
下面两个组件相互为root节点，循环引用。查看了下官网的示例。子菜单下有组，组下可以有子菜单
ElMenuGroupEx.vue
```javascript
<template>
  <el-menu-item-group v-if="menu.type == 'group'">
    <template slot="title"><i :class="menu.icon"></i>{{ menu.lable }}</template>
    <template v-if="menu.childs != null">
      <template v-for="menuItem in menu.childs">
        <el-menu-item-ex :menu="menuItem"></el-menu-item-ex>
        <el-menu-submenu-ex :menu="menuItem"></el-menu-submenu-ex>
      </template>
    </template>
  </el-menu-item-group>
</template>

<script>
  import ElMenuItemEx from './ElMenuItemEx.vue'

  export default {
    name: 'el-menu-group-ex',
    props: {
      menu: {
        type: Object
      }
    },
    components: {
      ElMenuItemEx
    },
    beforeCreate: function () {
//          递归组件
      //      https://cn.vuejs.org/v2/guide/components.html#组件间的循环引用Circular-References-Between-Components
      this.$options.components.ElMenuSubmenuEx = require('./ElMenuSubmenuEx.vue')
    },
    data () {
      return {}
    }
  }
</script>
<style lang="less" rel="stylesheet/less">

</style>

```
ElMenuSubmenuEx.vue
```javascript

<template>
  <el-submenu :index="menu.index" v-if="menu.type == 'submenu'">
    <template slot="title" v-if="menu.lable != null">
      <i :class="menu.icon"></i>{{ menu.lable }}
    </template>
    <template v-if="menu.childs != null">
      <template v-for="menuItem in menu.childs">
        <el-menu-group-ex :menu="menuItem"></el-menu-group-ex>
        <el-menu-item-ex :menu="menuItem"></el-menu-item-ex>
      </template>
    </template>
  </el-submenu>
</template>

<script>
  import ElMenuGroupEx from './ElMenuGroupEx.vue'
  import ElMenuItemEx from './ElMenuItemEx.vue'

  export default {
    name: 'el-menu-submenu-ex',
    props: {
      menu: {
        type: Object
      }
    },
    components: {ElMenuGroupEx, ElMenuItemEx},
    data () {
      return {}
    }
  }
</script>
<style lang="less" rel="stylesheet/less">

</style>

```

与官方对应item的组件处理完了。但是还不能直接使用。还有一个最外层的组件。这个组件的处理方法也是一样，不过不能公用了。因为里面还需要取数据，定制外观等各种配置操作。所以想了一个折中的办法。封装成系统业务组件。
TlzMenu.vue
```javascript
<template>
  <el-menu theme="dark" @select="handleSelect" :unique-opened="true">
    <template v-for="menuItem in leftMenus">
      <el-menu-group-ex :menu="menuItem"></el-menu-group-ex>
      <el-menu-item-ex :menu="menuItem"></el-menu-item-ex>
      <el-menu-submenu-ex :menu="menuItem"></el-menu-submenu-ex>
    </template>
  </el-menu>
</template>

<script>
  import ElMenuGroupEx from '../../comm/menu/ElMenuGroupEx.vue'
  import ElMenuItemEx from '../../comm/menu/ElMenuItemEx.vue'
  import ElMenuSubmenuEx from '../../comm/menu/ElMenuSubmenuEx.vue'

  export default {
    data () {
      return {
        leftMenus: []
      }
    },
    components: {ElMenuGroupEx, ElMenuItemEx, ElMenuSubmenuEx},
    created () {
      this.mockData()
    },
    methods: {
      handleSelect (key, keyPath, item) {
        if (item.route) {
          if (item.route.type === 'inner') {
            this.$router.push(item.route.path)
          }
        }
        console.log(key, keyPath)
      },
      mockData () {
        /**
         * 饿了么动态导航渲染参数说明：
         * type: 'group' ：类型：支持三种：对应官网组件名称：submenu,item,group
         index: '1-1', : 对应官网属性
         lable: '分组1' ：菜单名称
         icon: 'el-icon-message' ： 菜单图标，使用class名称
         childs:[] : 子元素，数组组成
         route: {  路由。 当type = submenu,group 的时候该属性无效
                type: 'inner',  路由跳转方式，
                                inner: 主要由绑定的handleHeaderCommand函数中去判断路由跳转
                                out: 由子组件中去处理成a标签链接，直接跳转到外部网站
                path: '/login'  跳转地址
          }
         */
        this.leftMenus.push({
          type: 'submenu',
          index: '1',
          lable: '导航一',
          icon: 'el-icon-message',
          childs: [
            {
              type: 'group',
              index: '1-1',
              lable: '分组1',
              icon: 'el-icon-message',
              childs: [
                {
                  type: 'item',
                  index: '1-1-1',
                  lable: '选项1',
                  icon: 'el-icon-message',
                  route: {
                    type: 'inner',
                    path: '/login'
                  }
                },
                {
                  type: 'item',
                  index: '1-1-2',
                  lable: '选项2',
                  icon: 'el-icon-message',
                  route: {
                    type: 'inner',
                    path: '/login'
                  }
                },
                {
                  type: 'submenu',
                  index: '1-1-3',
                  lable: '选项3',
                  icon: 'el-icon-message',
                  childs: [
                    {
                      type: 'group',
                      index: '1-1-3',
                      lable: '分组',
                      childs: [
                        {
                          type: 'item',
                          index: '1-1-3-1',
                          lable: '选项1',
                          icon: 'el-icon-message',
                          route: {
                            type: 'inner',
                            path: '/login'
                          }
                        },
                        {
                          type: 'item',
                          index: '1-1-3-2',
                          lable: '选项1',
                          icon: 'el-icon-message',
                          route: {
                            type: 'inner',
                            path: '/login'
                          }
                        }
                      ]
                    },
                    {
                      type: 'item',
                      index: '2',
                      lable: '菜单动态测试',
                      route: {
                        type: 'inner',
                        path: '/login'
                      }
                    }
                  ]
                }
              ]
            },
            {
              type: 'item',
              index: '1-2',
              lable: '分组同级选项1',
              route: {
                type: 'inner',
                path: '/login'
              }
            }
          ]
        })
        this.leftMenus.push({
          type: 'item',
          index: '2',
          lable: '选项2',
          route: {
            type: 'inner',
            path: '/login'
          }
        })
        this.leftMenus.push({
          type: 'group',
          index: '3',
          lable: '分组3',
          childs: [
            {
              type: 'item',
              index: '3-1',
              lable: '选项1',
              icon: 'el-icon-message',
              route: {
                type: 'inner',
                path: '/login'
              }
            },
            {
              type: 'item',
              index: '3-2',
              lable: '选项2',
              icon: 'el-icon-message',
              route: {
                type: 'inner',
                path: '/login'
              }
            },
            {
              type: 'submenu',
              index: '3-3',
              lable: '子菜单1',
              icon: 'el-icon-message',
              childs: [
                {
                  type: 'group',
                  index: '3-3-1',
                  lable: '分组',
                  childs: [
                    {
                      type: 'item',
                      index: '3-3-1-1',
                      lable: '选项1',
                      icon: 'el-icon-message',
                      route: {
                        type: 'inner',
                        path: '/login'
                      }
                    },
                    {
                      type: 'item',
                      index: '3-3-1-2',
                      lable: '选项2',
                      icon: 'el-icon-message',
                      route: {
                        type: 'inner',
                        path: '/login'
                      }
                    }
                  ]
                }
              ]
            }
          ]
        })
        this.leftMenus.push({
          type: 'item',
          index: '4',
          lable: '网站外部跳转',
          icon: 'el-icon-message',
          route: {
            type: 'out',
            path: 'http://www.baidu.com'
          }
        })
      }
    }
  }
</script>

<style lang="less" rel="stylesheet/less" scoped>

</style>

```
