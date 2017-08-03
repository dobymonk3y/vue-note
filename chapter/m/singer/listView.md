![](assets/musicapp/歌手列表快速列表样式.png)# listView歌手列表联/系人列表


## 第一个版本
![](/assets/musicapp/listView歌手列表联-系人列表.png)

上面的列表使用了scoll组件，然后渲染了列表
```html
<template>
  <scroll class="list-view" :data="data">
    <ul>
      <li v-for="group in data" class="list-group">
        <h2 class="list-group-title">{{ group.title }}</h2>
        <ul>
          <li v-for="item in group.items" class="list-group-item">
            <!-- 使用了图片的懒加载 --> 
            <img v-lazy="item.avtar" class="avtar">
            <span class="name">{{item.name}}</span>
          </li>
        </ul>
      </li>
    </ul>
  </scroll>
</template>
```
```javascript
<script type="text/ecmascript-6">
  import Scroll from 'base/scroll/Scroll'

  export default {
    props: {
      data: {
        type: Array,
        default: []
      }
    },
    components: {
      Scroll
    },
    data () {
      return {}
    }
  }
</script>
```
```css
<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .list-view {
    width: 100%
    height: 100%
    overflow hidden //这个应该写在父级里面限制这个listview的把。没有明白为什么写在这里也有效果
    background $color-background
    .list-group {
      padding-bottom 30px
      .list-group-title {
        font-size $font-size-small
        line-height 30px
        padding-left 20px
        color $color-text-l
        background $color-highlight-background
      }
      .list-group-item {
        display flex
        align-items center
        padding 20px 0 0 30px
        .avtar {
          width: 50px
          height: 50px
          border-radius: 50%
        }
        .name {
          margin-left 20px
          color $color-text-l
          font-size $font-size-medium
        }
      }
    }
  }
</style>
```

## 右侧快速入口开发
右侧的快速入口，是一个title列表。

![](/assets/musicapp/歌手列表快速列表样式.png)

使用计算属性从列表中拿到所有的title，组成集合
```javascript
    computed: {
      // 快速入口列表集合
      shortcutList () {
        return this.data.map((group) => {
          return group.title.substring(0, 1)  // 热门有两个字，所以要截取掉
        })
      }
    },
```
列表渲染
```html
    <div class="list-shortcut">
      <ul>
        <li class="item" v-for="item in shortcutList">{{ item }}</li>
      </ul>
    </div>
```
```css
.list-shortcut {
      position absolute
      right: 0
      top: 50%
      transform: translateY(-50%)
      width: 20px
      padding 20px 0
      background: $color-background-d
      border-radius: 10px
      text-align: center
      font-family: Helvetica
      .item {
        padding 3px
        color: $color-text-l
        font-size: $font-size-small
        line-height 1
        &.current {
          color: $color-theme
        }
      }
    }
```

## 右侧快速入口 点击让列表滚动到指定的title处

我们来分析下这个需求有哪些关键点

1. 快速入口中的每一个title都需要有一个点击事件
2. 让左侧的歌手列表滚动到相应的title处

我们先来解决第一个：点击事件,将h5的触摸事件设置在快速入口的元素上，@touchstart，

```html
<div class="list-shortcut" @touchstart="onShortcutTouchStart">
      <ul>
        <li class="item" v-for="(item,index) in shortcutList"
            :data-index="index">{{ item }}
        </li>
      </ul>
    </div>
```
那么这个元素中的任何一个元素被触摸的时候都会触发该事件。所以这里就有一个问题，怎么才能捕获到只触摸文字的时候（也就是class=item的时候）才触发滚动操作？
```javascript
    methods: {
      // H5 的触摸开始事件
      onShortcutTouchStart (el) {
        // 拿到dom元素在列表中的索引
        let anchorIndex = getData(el.target, 'index')
        if(anchorIndex){
          this.$refs.listview.scrollToElement(this.$refs.listgroup[anchorIndex])
        }
      }
    },
---- getData ----
/**
 * 获取dom 元素上的属性，data开头的属性，
 * @param el
 * @param name
 * @param val 有值就设置，没有值就获取
 * @returns {*}
 */
export function getData (el, name, val) {
  const prefix = 'data-'
  name = prefix + name
  if (val) {
    return el.setAttribute(name, val)
  } else {
    return el.getAttribute(name)
  }
}
```

通过上面的源码可以看到，我们获取触发事件中的dom元素上的'data-index' 属性（在遍历的时候设置在每一个入口文字上的），如果有，就表示需要左侧的内容需要滚动到对应的区域。

