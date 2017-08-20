# 歌手详情页-music-list组件

歌曲列表组件开发，还要啰嗦一句，第一次开发你肯定不知道哪些是需要组件的，所以这个组件的抽象，要么是做之前，看了一个整体的设计图，然后分离出来的，观察他们的表现形式，是否类似，结构是否类似，如果是的话就可以做成组件了。

## 骨架页
![](/assets/musicapp/歌手详情歌曲列表页面骨架效果.png)

先分析下页面结构和实现思路：

1. 一个返回按钮
2. 一个标题
3. 一个背景图
4. 一个歌曲列表

实现思路：

1. 返回按钮 ：小图标加绝对定位
2. 标题 ：绝对定位 + 宽度百分之80 + left 10% + 文字居中，正好让标题块居中
3. 背景图 ： background-image:url() + 一个div模拟的半透明滤镜 + 绝对定位覆盖掉背景图片

上面html和css要注意的是，因为dom结构的顺序，所以要注意z-index 的值，因为后来居上，所以要设置z-index的值不然就看不到返回按钮和标题了

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

## 列表的页的引用
引入 [歌手详情页-song-list组件的 - 编写列表组件 小节中的第一个版本列表](/chapter/m/singer/detail_song_list.md)
因为还用到了滑动，所以还要引入基础组件的scroll组件

```html
.....
<scroll :data="songs" class="list" ref="list">
      <div class="song-list-wrapper">
        <song-list :songs="songs"></song-list>
      </div>
    </scroll>
```
```javascript
script type="text/ecmascript-6">
  import SongList from 'base/song-list/song-list'
  import Scroll from 'base/scroll/Scroll'

  export default {
    components: {
      Scroll, SongList
    },
    ......
    mounted () {
      // 因为这是一个scroll组件，所以要拿到el（dom元素）
      // 为什么要动态计算高度呢？是因为背景图的高度是自适应的
      this.$refs.list.$el.style.top = `${this.$refs.bgImage.clientHeight}px`
      console.log(this.$refs.list)
    }
```
```css
....
 .list {
      position: fixed
      top: 0
      bottom: 0
      width: 100%
      background: $color-background
      overflow hidden
      .song-list-wrapper {
        padding: 20px 30px
      }
      .loading-container {
        position: absolute
        width: 100%
        top: 50%
        transform: translateY(-50%)
      }
    }
```


死板的列表样子是做出来的。
接下来我们要实现一个动效（类似原生的app交互效果）：

1. 希望在列表往下滑动（手指会往上滑动）的时候，上面的背景图局域能随着滑动的上升而挤走
2. 列表往上滑动（手指会往下滑动）的时候，背景图随着滑动的下降而逐渐显示出来

## 交互效果
先来手指往上滑动，歌曲列表会滚动到距离页面顶部的一定距离处，然后固定

实现思路：

1. 去掉了list的overflow hidden，能让我们滚动出容器，但是容器背景不会跟着滚动
2. 所以我们需要另外一个层与容器背景颜色一致的，滚动的时候把这个层跟着滚动，看起来的效果就是这个列表往上移动了。
  
    为什么能让这个层跟着滚动还能做列表的背景呢？因为css样式，列表是position: fixed布局，top是js根据上面的背景图片部分计算出来的。所以是悬浮的。而层是普通的div元素。使用translate3d让div元素偏移。


