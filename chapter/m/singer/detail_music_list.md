# 歌手详情页-music-list组件

歌曲列表组件开发，还要啰嗦一句，第一次开发你肯定不知道哪些是需要组件的，所以这个组件的抽象，要么是做之前，看了一个整体的设计图，然后分离出来的，观察他们的表现形式，是否类似，结构是否类似，如果是的话就可以做成组件了。

## 骨架页
![](/assets/musicapp/歌手详情歌曲列表页面骨架效果.png)

**src/components/music-list/music-list.vue**
```html
<template>
  <div class="music-list">
    <div class="back">
      <i class="icon-back"></i>
    </div>
    <h1 class="title" v-html="title"></h1>
    <div class="bg-image" :style="bgStyle">
      <div class="filter"></div>
    </div>
  </div>
</template>
```
```javascript
<script type="text/ecmascript-6">
  export default {
    data () {
      return {}
    },
    props: {
      // 背景图
      bgImage: {
        type: String,
        default: ''
      },
      // 歌曲列表
      songs: {
        type: Array,
        default: []
      },
      // 歌手名称
      title: {
        type: String,
        default: ''
      }
    },
    computed: {
      bgStyle () {
        return `background-image:url(${this.bgImage})`
      }
    }
  }
</script>
```
```css
<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"
  @import "~common/stylus/mixin"

  .music-list {
    position fixed
    z-index 100
    top 0
    left 0
    bottom 0
    right 0
    background $color-background
    .back {
      position absolute
      top: 0
      left: 6px
      z-index: 50
      .icon-back {
        display block
        padding 10px
        font-size $font-size-large-x
        color: $color-theme
      }
    }
    .title {
      position absolute
      top 0
      left 10% // 宽度百分之80，左边10，右边10也就居中了
      width 80%
      z-index 40
      no-wrap()
      text-align center
      line-height 40px
      font-size $font-size-large
      color $color-text
    }
    .bg-image {
      position relative
      width 100%
      height 0
      padding-top 70%
      transform-origin top
      background-size cover
    // 滤镜效果，图片用背景做
    // 滤镜用div定位做
      .filter {
        position absolute
        top 0
        left 0
        width 100%
        height 100%
        background rgba(7, 17, 27, 0.4)
      }
    }
  }
</style>
```