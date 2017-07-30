# 轮播图组件-slider
### 轮播图实现
需要依赖 https://github.com/ustbhuangyi/better-scroll 库，移动端的滑动库？
```bash
npm i better-scroll -S

```
```html
<template>
  <div class="slider" ref="slider">
    <div class="slider-group" ref="sliderGroup">
      <slot></slot>
    </div>
    <div class="dots">
      <span class="dot" v-for="(item,index) in dots"
            :class="{active:currentPageIndex === index}"
      ></span>
    </div>
  </div>
</template>
```
```javascript
<script type="text/ecmascript-6">
  /**
   * 轮播图组件的思路
   * 1. 轮播图的组成
   *    1. 图片元素的滚动
   *    2. 小圆点提示 （dot）
   * 2. 把样式写出来
   * 3. 轮播图的原理是：在一行中，滚动横向的滚动条，达到滚动的效果，所以需要对总宽度和每个元素的宽度用js计算
   * 4. dot的对应提示，现在是第几张图片则让对应的小圆点激活
   * 5. 自动播放，利用better-scroll的goToPage编程式的触发滚动到下一页
   *    1. 但是需要注意计算无缝切换和常规切换的元素索引问题
   *    2. 还要考虑到定时器过程中如果手动触发了滚动，那么要在调用定时滚动到下一页前进行清空定时器
   * 6. 窗口大小改变的话，还需要监听resize事件，重新设置_setSliderWidth的宽度
   */
  import BScroll from 'better-scroll'
  import { addClass } from 'common/js/dom.js'
  export default {
    props: {
      loop: {  // 是否可以无缝轮播，也就是从头滚到后，再往后就由滚到头了
        type: Boolean,
        default: true
      },
      autoPlay: { // 是否自动轮播
        type: Boolean,
        default: true
      },
      interval: {  // 轮播间隔时间，毫秒
        type: Number,
        default: 4000
      }
    },
    mounted () {
      // 为了能正确渲染
      // 浏览器刷新一次是17毫秒，所以设置成20毫秒比较合理
      setTimeout(() => {
        this._setSliderWidth()
        this._initdots()
        this._initSlider()
      }, 20)

      window.addEventListener('resize', () => {
        if (this.slider) {
          this._setSliderWidth(true)
          this.slider.refresh()
        }
      })
    },
    methods: {
      // 横向滚动，需要设置一个宽度
      /**
       * @param isResize 是否是重新设置宽度,因为初始化的时候把宽度增加了2个元素的，
       * 但是在循环获取轮播元素的时候，第一次BScroll没有实例化，所以元素个数没有那么多，被我们增加了2个元素的宽度，
       * 如果是重置的话，那么就不应该再增加两个元素的宽度了
       * */
      _setSliderWidth (isResize) {
        // 横向滚动，宽度不能确定，所以要计算宽度
        let children = this.$refs.sliderGroup.children
//        console.log(children)
        this.children = children
        let width = 0   // 总宽度
        // 该元素被图片宽度撑开，能获取到宽度
        let sliderWidth = this.$refs.slider.clientWidth
        /* TODO: 这里渲染会有一个问题，因为一般数据都是异步的，这里是组件创建就进行初始化，
         所以需要外部使用的地方保证组件初始化的时候插槽内的元素被填充
         */
        for (let i = 0; i < this.children.length; i++) {
          // 对插槽内的图片元素处理
          // 样式也需要这里做处理
          let child = this.children[i]
          addClass(child, 'slider-item')
          child.style.width = sliderWidth + 'px'
          width += sliderWidth
//          console.log(width)
        }

        // 无缝滚动的话，BScroll 会把头尾的都copy一份追加，所以多增加两倍的宽度
        if (this.loop && !isResize) {
          width += 2 * sliderWidth
        }
        // 因为是滚动，所以需要让待滚动的元素排列成一排，视窗大小只有手机屏幕那么大
        // 加上滑动效果，就能让当前的div元素横向滚动
        // 这个应该就是轮播图的原理，复杂的就是怎么滚动适合的宽度，所以使用了BScroll吗/
        this.$refs.sliderGroup.style.width = width + 'px'
      },
      // 在初始化BScroll之前要确认 小圆点的数量，因为BScroll会把滚动元素多增加两个
      _initdots () {
        this.dots = new Array(this.children.length)
      },
      _initSlider () {
        this.slider = new BScroll(this.$refs.slider, {
          scrollX: true,   // 滚动方向为 X 轴
          scroolY: false,  // 滚动方向为 Y 轴
          momentum: true,  // 当快速滑动时是否开启滑动惯性
          // click: true,  // 该属性打开的话，用a标签跳转的就会失效了，该属性的作用是，在轮播图上面点击BScroll会派发一个自定义的事件，而忽略了a标签的跳转事件（或则是和fastClick库冲突）
          snap: true, // 该属性是给 slider 组件使用的，普通的列表滚动不需要配置
          snapLoop: this.loop, // 是否可以无缝循环轮播
          snapThreshold: 0.1,  // 用手指滑动时页面可切换的阈值，大于这个阈值可以滑动的下一页
          snapSpeed: 400  // 轮播图切换的动画时间
        })
        this.slider.on('scrollEnd', () => {
          // 获取x轴的当前元素索引
          let pageIndex = this.slider.getCurrentPage().pageX
//          console.log('currentPageIndex:', pageIndex)
          // 如果是无缝切换的话，因为在首尾都copy了一份，所以要处理这个
          // 但是 pageIndex 从BScroll中反馈回来的是按照增加之后的元素索引反馈的
          // 元素索引是从0开始，因为收尾增加了一项，所以默认BScroll默认显示是真正的第一页
          // 也就是我们所想看到的第一页，但是在BScroll中的索引是 1，0 是BScroll框架增加的哪一个元素
          if (this.loop) {
            pageIndex--
          }
          this.currentPageIndex = pageIndex
          if (this.autoPlay) {
            clearTimeout(this.timer) // 先清空定时器,不然已经触发还没到时间的，还会触发
            this._play()
          }
        })
        if (this.autoPlay) {
          this._play()
        }
      },
      _play () {
        // 滚动到下一张图片
        let pageIndex = this.currentPageIndex + 1
        if (this.loop) {
          pageIndex++   // 如果是无缝切换的话需要把索引+1
        }
        this.timer = setTimeout(() => {
          this.slider.goToPage(pageIndex, 0, 400)
          // this._play() 不能在这里进行调取，因为如果自动的过程中手动拖动了，那么正好定时器到了，就会被立马切换
          // 所以需要在 切换的时候调取
        }, this.interval)
      }
    },
    data () {
      return {
        children: [],  // 插槽元素数量，也就是滚动图片的item
        slider: null, // BScroll实例
        dots: [],
        currentPageIndex: 0,  // 当前是第几个元素
        timer: null  // 自动轮播滚动定时器
      }
    },
    destroyed () {
      // 该组件实例被销毁后调用，清空计时器
      clearTimeout(this.timer)
    }
  }
</script>
```
```css
<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"
  .slider
    min-height: 1px
    position: relative
    width: 100%
    overflow: hidden
    .slider-group
      position relative
      overflow hidden
      white-space nowrap
      .slider-item
        float: left
        box-sizing: border-box
        overflow: hidden
        text-align: center
        a
          display block
          width 100%
          overflow hidden
          text-decoration: none
        img
          display: block
          width: 100%
    .dots
      position absolute
      right 0
      left 0
      text-align center
      font-size 0
      bottom: 12px
      .dot
        display: inline-block
        margin: 0 4px
        width: 8px
        height: 8px
        border-radius: 50%
        background: $color-text-l
        &.active
          width: 20px
          border-radius: 5px
          background: $color-text-ll
</style>

```


