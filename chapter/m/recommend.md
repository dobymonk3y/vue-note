# 推荐页面 + jsonp获取数据

## 数据接口调研
在谷歌浏览器中切换到手机模式，访问 `https://m.y.qq.com/`，然后查看请求，找到需要的请求链接，因为是抓去别人的数据，所以人家获取数据的方式不是一层不变的。 请求过滤：
1. xhr 是ajax请求
2. js 是获取的一些js文件，也包括跨域数据，这就要说道jsop原理了

这次获取到的是该地址返回的页面所需要的数据：`https://c.y.qq.com/musichall/fcgi-bin/fcg_yqqhomepagerecommend.fcg?g_tk=5381&uin=0&format=json&inCharset=utf-8&outCharset=utf-8&notice=0&platform=h5&needNewCode=1&_=1498359279556`

视频中是jsonp访问的。而我看的时候获取到的是同源访问，那么联想到视频中抓去到请求参数`jsonpCallback=jsonp1`,尝试加了该参数，访问的数据就是以json1{xxx} 的形式返回的了。很厉害，一个接口支持所有的格式返回


注意：在抓包获取接口的时候，有可能会遇到特殊处理的比如；pc站的qq音乐的歌单列表
```
https://y.qq.com/portal/playlist.html#t2=5&
上面是站点页面，下面是歌单列表地址，直接请求都显示无法显示网页，但是刷新页面能看到。这样的就是做了一些特殊处理的。
https://c.y.qq.com/splcloud/fcgi-bin/fcg_get_diss_by_tag.fcg?rnd=0.8914799780923608&g_tk=5381&jsonpCallback=getPlaylist&loginUin=0&hostUin=0&format=jsonp&inCharset=utf8&outCharset=utf-8&notice=0&platform=yqq&needNewCode=0&categoryId=10000000&sortId=5&sin=0&ein=29
```

下面搜集一些处理方式：
### 比如上面的请求链接
```
在请求头里面有1个属性： 
referer:https://y.qq.com/portal/playlist.html

该字段暂时无法处理的。使用开发环境的代理功 + axios 设置请求头来模拟正确的请求头
```

在配置文件dev-server.js中的express
```javascript
var axios = require('axios')
var apiRoutes = express.Router()

apiRoutes.get('/getDiscList', function (req, res) {
  var url = 'https://c.y.qq.com/splcloud/fcgi-bin/fcg_get_diss_by_tag.fcg'
  axios.get(url, {
    headers: {
      referer: 'https://c.y.qq.com/',
      host: 'c.y.qq.com'
    },
    params: req.query    // 代理到的查询参数
  }).then((response) => {
    res.json(response.data)   // 代理获取后返回的res
  }).catch((e) => {
    console.log(e)
  })
})

app.use('/api', apiRoutes)

在调用的时候，就不是jsonp请求了，而是一个普通的ajax请求了。
```



## jsonp
https://github.com/webmodules/jsonp

> 很简单，就是利用`<script>`标签没有跨域限制的“漏洞”（历史遗迹啊）来达到与第三方通讯的目的。
 当需要通讯时，本站脚本创建一个`<script>`元素，地址指向第三方的API网址，形如： `<script src="http://www.example.net/api?param1=1&param2=2"></script>`     并提供一个回调函数来接收数据（函数名可约定，或通过地址参数传递）。     
 第三方产生的响应为json数据的包装（故称之为jsonp，即json padding），形如：     `callback({"name":"hax","gender":"Male"})`     这样浏览器会调用`callback`函数，并传递解析后json对象作为参数。本站脚本可在`callback`函数里处理所传入的数据。    
补充：“历史遗迹”的意思就是，如果在今天重新设计的话，也许就不会允许这样简单的跨域了嘿，比如可能像XHR一样按照CORS规范要求服务器发送特定的http头。

把该库安装到项目中
```bash
npm i jsonp -S

```

因为很多地方都会用到，所以适当的进行工具类的封装
```javascript
import originJSONP from 'jsonp'

export default function jsonp (url, data, option) {
  // 需要判断url后面是否有问号
  url += (url.indexOf('?') < 0 ? '?' : '&') + param(data)
  return new Promise((resolve, reject) => {
    originJSONP(url, option, (err, data) => {
      if (!err) {
        resolve(data)
      } else {
        reject(err)
      }
    })
  })
}

function param (data) {
  let url = ''
  for (var k in data) {
    let value = data[k] !== undefined ? data[k] : ''
    url += `&${k}=${encodeURIComponent((value))}`
  }
  // 如果url有值，则将第一个 & 符号去掉
  return url ? url.substring(1) : ''
}

```

## 流程
1. 先把需要抓去的数据拿到
2. 按照页面一个一个来解决
3. 先解决轮播图，编写轮播图的基础组件 base/slider

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


### SCROLL组件的抽象和应用

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



