# Scroll组件的抽象和应用

列表现在是不能滚动的，用原生的pc端的滚动的话，滚动条会很丑，且还没有一些动量什么的效果
```html
<template>
  <div ref="wrapper">
    <slot></slot>
  </div>
</template>
```
```javascript
<script type="text/ecmascript-6">
  import BScroll from 'better-scroll'

  export default {
    props: {
      /**
       * 1 会截流,只有在滚动结束的时候派发一个 scroll 事件。
       * 2 在手指 move 的时候也会实时派发 scroll 事件，不会截流。
       * 3 除了手指 move 的时候派发scroll事件，在 swipe（手指迅速滑动一小段距离）的情况下，列表会有一个长距离的滚动动画，这个滚动的动画过程中也会实时派发滚动事件
       */
      probeType: {
        type: Number,
        default: 1
      },
      // true 是否派发click事件
      click: {
        type: Boolean,
        default: true
      },
      // 数据是动态的，数据变了就要重新刷新bascroll否则不能滚动
      // 要重新计算dom的高度
      // 这里必须安利下：很多时候scroll包裹的dom元素的高度都是动态数据撑开的
      // 所以需要在数据变化后，进行重新刷新scroll，才能实现滚动
      data: {
        type: Array,
        default: null
      }
    },
    mounted () {
      // 为了能正确渲染
      // 浏览器刷新一次是17毫秒，所以设置成20毫秒比较合理
      setTimeout(() => {
        this._initScrool()
      }, 20)
    },
    methods: {
      _initScrool () {
        if (!this.$refs.wrapper) {
          return
        }
        this.scroll = new BScroll(this.$refs.wrapper, {
          probeType: this.probeType,
          click: this.click
        })
      },
      // 下面代理几个比较常用的方法
      enable () {
        this.scr && this.scroll.enable()
      },
      disable () {
        this.scr && this.scroll.disable()
      },
      refresh () {
        this.scroll && this.scroll.refresh()
      }
    },
    data () {
      return {}
    },
    watch: {
      data () {
        setTimeout(() => {
          this.refresh()
        }, 20)
      }
    }
  }
</script>
```

在使用这个组件的时候，有一个问题，就是：

1. 如果包裹的里面有多个ajax请求数据渲染，那么最后一个ajax回来的时候刷新该组件也有能正确滚动，如果不知道哪一个先回来呢？那么就需要每个回来的时候都需要刷新。

2. 如果里面有图片的话，图片会下载，下载完成之后高度才会被撑开。所以图片加载也需要刷新

  ```
  // 给图片添加load事件，然后再该事件中
  <img @load="loadImage" :src="item.picUrl">
  ```



