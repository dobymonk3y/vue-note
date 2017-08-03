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