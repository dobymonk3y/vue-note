![](assets/musicapp/歌手列表组件固定标题悬滚动中的细节效果优化.png)# listView歌手列表联/系人列表 2

前面基本上把轮廓功能都弄好了。接下来应该是优化了； 用好 bscroll很重要，一些事件什么时候派发有什么特性直接影响了组件的功能效果；完整代码如下：

```javascript
<template>
  <scroll class="list-view"
          ref="listview"
          :data="data"
          :listen-scroll="true"
          :probe-type="3"
          @scroll="listScroll">
    <ul>
      <li v-for="group in data" class="list-group" ref="listGroup">
        <h2 class="list-group-title">{{ group.title }}</h2>
        <ul>
          <li v-for="item in group.items" class="list-group-item">
            <img v-lazy="item.avtar" class="avtar">
            <span class="name">{{item.name}}</span>
          </li>
        </ul>
      </li>
    </ul>
    <div class="list-shortcut" @touchstart="onShortcutTouchStart"
         @touchmove.stop.prevent="onShortcutTouchMove">
      <ul>
        <li class="item"
            v-for="(item,index) in shortcutList"
            :class="{'current':currentIndex == index}"
            :data-index="index">{{ item }}
        </li>
      </ul>
    </div>
  </scroll>
</template>

<script type="text/ecmascript-6">
  import Scroll from 'base/scroll/Scroll'
  import { getData } from 'common/js/dom'

  const ANCHOR_HEIGHT = 18  // 每个 list-shortcut 中 item的高度，18=item的字体12 + pading上下3*2

  export default {
    created () {
      // 为什么不在 data里面定义touch？
      // 因为在props中和data中定义的都会被vue监听用于数据的双向绑定
      // 这个并不需要这些监听服务，只需要在当前vue实例中共享数据
      this.touch = {}

      // 配合_calculateHeight方法 存储计算的高度
      this.listHeight = []
    },
    props: {
      data: {
        type: Array,
        default: []
      }
    },
    components: {
      Scroll
    },
    computed: {
      // 快速入口列表集合
      shortcutList () {
        return this.data.map((group) => {
          return group.title.substring(0, 1)  // 热门有两个字，所以要截取掉
        })
      }
    },
    methods: {
      // H5 的触摸开始事件
      onShortcutTouchStart (el) {
        let firstTouch = el.touches[0]
        this.touch.y1 = firstTouch.pageY

        // 拿到dom元素在列表中的索引
        let anchorIndex = getData(el.target, 'index')
        if (anchorIndex) {
          this.touch.anchorIndex = anchorIndex
          this._scrollToElement(anchorIndex)
          console.log(anchorIndex)
        }
      },
      onShortcutTouchMove (e) {
        let firstTouch = e.touches[0]
        this.touch.y2 = firstTouch.pageY
        // y 轴上的偏移像素
        let delta = this.touch.y2 - this.touch.y1
        // 得到有几个元素
        delta = delta / ANCHOR_HEIGHT
        // 取整
        delta = delta | 0
        let anchorIndex = (parseInt(this.touch.anchorIndex) + delta)
        let length = this.$refs.listGroup.length
        if (anchorIndex >= length) {
          // 由于触摸和scroll事件不同，触摸是按照第一次触摸的点为起点，
          // 这个计算就会超过实际的索引大小
          anchorIndex = length - 1
        } else if (anchorIndex < 0) {
          anchorIndex = 0
        }

        this._scrollToElement(anchorIndex)
        console.log(delta, anchorIndex)
      },
      _scrollToElement (index) {
        // 由于滚动特性，顶部是0，以下的都是负数，不然在触发watch.scrollY的时候会被认定为滚动到了列表最顶端
        this.currentIndex = index
        this.$refs.listview.scrollToElement(this.$refs.listGroup[index], 0)  // 滚动速度，0 是没有滚动动画
      },
      listScroll (pos) {
        this.scrollY = pos.y
      },
      // 计算每个group的高度（数据结构是按组进行聚合的）
      _calculateHeight () {
        this.listHeight = []
        const list = this.$refs.listGroup
        let height = 0
        this.listHeight.push(height)
        for (let i = 0; i < list.length; i++) {
          let item = list[i]
          height += item.clientHeight  // 计算每一个group所在的高度区域，从0开始
          this.listHeight.push(height)
        }
      }
    },
    data () {
      return {
        // 监控y轴的位置
        scrollY: -1,
        // 当前索引
        currentIndex: 0
      }
    },
    watch: {
      data () {
        setTimeout(() => {
          this._calculateHeight()
        }, 20)
      },
      // 根据滚动的高度计算当前滚动到的索引，然后利用索引就能得到右侧的索引元素
      scrollY (newY) {
        // 滚动是从起始点顶点开始，而触摸滚动是从第一次触点算做起始点
        if (newY >= 0) {
          this.currentIndex = 0
          return
        }
        // 中部区域滚动
        let listHeight = this.listHeight
        for (let i = 0; i < listHeight.length; i++) {
          let height1 = listHeight[i]
          let height2 = listHeight[i + 1]
          let y = -newY // 滚动的话，向下滚动都是负数，从0开始
          // 1. height2 有可能下标越界，获得undefined
          // 2. 在一个区间内
          if (!height2 || (y > height1 && y < height2)) {
            this.currentIndex = i
            return
          }
        }
        // 尾部区域
        this.currentIndex = 0
      }
    }
  }
</script>

```

小结下该组件：

看起来组件比较简单，但是话费的时间比较多，主要在双向联动，任意一侧滚动都需要实时的联动。思路总结下

1. 核心js：better-scroll
2. 利用在右侧列表中使用h5的touchstart（触摸开始）来获得滚动起始点的容器y轴的像素。

  且能获取到当前触发的dom元素，从而拿到当前右侧列表点击的dom元素在列表中的索引（渲染列表的时候把索引加到dom上的）
3. 利用在右侧使用话的touchmove.stop.prevent（滚动事件，且用vue提供的阻止冒泡）获取当前滚动到的y轴像素。

  从而能计算出从起始点到滚动的当前点偏移的像素，而右侧列表的每个元素高度都是固定的，从而能换算出滚动了几个dom元素，然后起始点的索引加上算出来的，得到了当前的索引，进而利用该索引找到右侧内容的dom，再滚动到该dom
4. 右侧滚动事件是利用了bscroll的滚动事件派发，因为使用了基础组件的滚动组件。

  bscroll派发的滚动事件，传回的参数是当前元素滚动到的位置（是相对于组件的高度，最上面就是0，往下滚动就是负数的dom元素高度），所以我们要在数据被填充后，计算每个dom的高度，再通过回传的负数转为绝对值和计算的高度列表中的前一个dom元素和后一个dom元素比较区间，就能得到当前的滚动位置是哪一个dom元素，能拿到这个索引，就能让右侧联动滚动了。
  
综合以上4点的原理，这个dom所在的索引，是这个组件的核心算法。  


## 左侧滚动时，怎么固定左侧的标题在顶部展示
需求：左侧滚动的时候，需要当前区域的title固定在顶部（场景：如果左侧一个title的列表很多的时候，能知道当前title）

![](/assets/musicapp/歌手列表组件固定标题悬浮效果.png)

思路：

1. 左侧和右侧滚动都需要变更顶部的文案，那么还是依赖之前算出来的currentIndex
2. 悬浮效果呢？是一直有一个悬浮的框和列表组里面的框一样，一直悬浮在上面的（由于和列表组里面的一样，高度也一样，看起来就e很难玄妙）

变动的代码如下
```html
    <!--固定悬浮标题效果,此代码位置就在列表代码同级-->
    <div class="list-fixed">
      <h1 class="fixed-title">{{ fixedTitle }}</h1>
    </div>
```
```javascript
    computed: {
      // 固定标题悬浮效果 实时计算
      fixedTitle () {
        return this.data[this.currentIndex] ? this.data[this.currentIndex].title : ''
      }
    },
```
```css
.list-fixed {
      position: absolute;
      top: 0;
      right: 0;
      left: 0;
      .fixed-title {
        font-size: 12px;
        line-height: 30px;
        padding-left: 20px;
        color: rgba(255, 255, 255, 0.5);
        background: #333;
      }
    }
```

上面的效果实现之后，发现滚动到最顶部的时候，会出现两个一样的边框，上面说道的，这个边框标题一直都存在，所以要处理滚动到最顶部（往下拉，顶部会被往下拉出一定的距离，这个效果就出现了）的时候我们隐藏掉这个悬浮边框。
```javascript
      // 固定标题悬浮效果 实时计算
      fixedTitle () {
        if (this.scrollY >= 0) {
          return ''
        }
        return this.data[this.currentIndex] ? this.data[this.currentIndex].title : ''
      }
```
可以看到这里只是返回了空串，也能达到效果，是因为上面的css代码，高度是文字撑开的，当没有文字的时候，高度为0，就相当于隐藏了


## 细节优化
![](/assets/musicapp/歌手列表组件固定标题悬滚动中的细节效果优化.png)

注意看上图中，在滚动的时候，重叠的时候很生硬，没有一点点过度效果，下面的滚动到0的时候，才会直接显示下面的title。需要做一点过度效果

思路：

1. 在左侧列表滚动的时候，计算下一个元素顶部距离滚动组件的顶部的距离
2. 监视这个距离，达到悬浮框的高度的时候，就表示他们开始碰撞了（挨在一起了）
3. 再改变悬浮框的偏移（使用 transform=translated3d(0,重叠像素)）

  这样看起来效果是往上滚动的时候，group的标题框会把上一个悬浮框网上顶

```html
    <!--固定悬浮标题效果-->
    <div class="list-fixed" ref="fixed">
      <h1 class="fixed-title">{{ fixedTitle }}</h1>
    </div>
```
重点部分
```javascript
    watch: {
      // 根据滚动的高度计算当前滚动到的索引，然后利用索引就能得到右侧的索引元素
      scrollY (newY) {
        .....
        let listHeight = this.listHeight
        for (let i = 0; i < listHeight.length - 1; i++) {
          if (!height2 || (y > height1 && y < height2)) {
            this.currentIndex = i
            .....

            // 计算当前滚动的区域的下一个group的dom元素顶部 距离滚动容器顶部的距离
            // 用来计算 过度效果的偏移像素
            this.diff = height2 + newY
            return
          }
        }
        // 尾部区域
        this.currentIndex = 0
      },
      diff (newDiff) {
        // 在碰撞范围内，返回重叠的重叠的像素，一偏移，效果就像被碰撞走的
        let fixedTop = (newDiff > 0 && newDiff < TITLE_HEIGHT) ? newDiff - TITLE_HEIGHT : 0

        console.log('fixedTop:', fixedTop, 'newDiff:', newDiff)
        // 当不在 TITLE_HEIGHT 范围内的时候，fixedTop 始终返回0.下面再把固定标题框移动到0的位置
        // 且不再继续改变，只有在碰撞范围内，再继续碰撞的效果
        if (this.fixedTop === fixedTop) {
          return
        }
        console.log('---fixedTop:', fixedTop, 'newDiff:', newDiff)
        this.fixedTop = fixedTop
        this.$refs.fixed.style.transform = `translate3d(0,${fixedTop}px,0)`
      }
```
  
## 最终的一个完整代码
```javascript
<template>
  <scroll class="list-view"
          ref="listview"
          :data="data"
          :listen-scroll="true"
          :probe-type="3"
          @scroll="listScroll">
    <ul>
      <li v-for="group in data" class="list-group" ref="listGroup">
        <h2 class="list-group-title">{{ group.title }}</h2>
        <ul>
          <li v-for="item in group.items" class="list-group-item">
            <img v-lazy="item.avtar" class="avtar">
            <span class="name">{{item.name}}</span>
          </li>
        </ul>
      </li>
    </ul>
    <div class="list-shortcut" @touchstart="onShortcutTouchStart"
         @touchmove.stop.prevent="onShortcutTouchMove" v-show="data.length">
      <ul>
        <li class="item"
            v-for="(item,index) in shortcutList"
            :class="{'current':currentIndex == index}"
            :data-index="index">{{ item }}
        </li>
      </ul>
    </div>
    <!--固定悬浮标题效果-->
    <div class="list-fixed" ref="fixed">
      <h1 class="fixed-title">{{ fixedTitle }}</h1>
    </div>
    <div class="loading-container" v-show="!data.length">
      <loading></loading>
    </div>
  </scroll>
</template>

<script type="text/ecmascript-6">
  import Scroll from 'base/scroll/Scroll'
  import { getData } from 'common/js/dom'
  import Loading from 'base/loading/Loading'

  const ANCHOR_HEIGHT = 18  // 每个 list-shortcut 中 item的高度，18=item的字体12 + pading上下3*2
  const TITLE_HEIGHT = 30 // 组标题高度，css样式计算出来的

  export default {
    components: {
      Scroll, Loading
    },
    created () {
      // 为什么不在 data里面定义touch？
      // 因为在props中和data中定义的都会被vue监听用于数据的双向绑定
      // 这个并不需要这些监听服务，只需要在当前vue实例中共享数据
      this.touch = {}

      // 配合_calculateHeight方法 存储计算的高度
      this.listHeight = []
    },
    props: {
      data: {
        type: Array,
        default: []
      }
    },
    computed: {
      // 快速入口列表集合
      shortcutList () {
        return this.data.map((group) => {
          return group.title.substring(0, 1)  // 热门有两个字，所以要截取掉
        })
      },
      // 固定标题悬浮效果 实时计算
      fixedTitle () {
        if (this.scrollY >= 0) {
          return ''
        }
        return this.data[this.currentIndex] ? this.data[this.currentIndex].title : ''
      }
    },
    methods: {
      // H5 的触摸开始事件
      onShortcutTouchStart (el) {
        let firstTouch = el.touches[0]
        this.touch.y1 = firstTouch.pageY
        console.log('h5:', firstTouch.pageY)
        // 拿到dom元素在列表中的索引
        let anchorIndex = getData(el.target, 'index')
        if (anchorIndex) {
          this.touch.anchorIndex = anchorIndex
          this._scrollToElement(anchorIndex)
          console.log(anchorIndex)
        }
      },
      onShortcutTouchMove (e) {
        let firstTouch = e.touches[0]
        this.touch.y2 = firstTouch.pageY
        console.log('h52:', firstTouch.pageY)
        // y 轴上的偏移像素
        let delta = this.touch.y2 - this.touch.y1
        // 得到有几个元素
        delta = delta / ANCHOR_HEIGHT
        // 取整
        delta = delta | 0
        let anchorIndex = (parseInt(this.touch.anchorIndex) + delta)
        let length = this.$refs.listGroup.length
        if (anchorIndex >= length) {
          // 由于触摸和scroll事件不同，触摸是按照第一次触摸的点为起点，
          // 这个计算就会超过实际的索引大小
          anchorIndex = length - 1
        } else if (anchorIndex < 0) {
          anchorIndex = 0
        }

        this._scrollToElement(anchorIndex)
        console.log(delta, anchorIndex)
      },
      _scrollToElement (index) {
        // 由于滚动特性，顶部是0，以下的都是负数，不然在触发watch.scrollY的时候会被认定为滚动到了列表最顶端
        this.currentIndex = index
        this.$refs.listview.scrollToElement(this.$refs.listGroup[index], 0)  // 滚动速度，0 是没有滚动动画
      },
      listScroll (pos) {
        this.scrollY = pos.y
      },
      // 计算每个group的高度（数据结构是按组进行聚合的）
      _calculateHeight () {
        this.listHeight = []
        const list = this.$refs.listGroup
        let height = 0
        this.listHeight.push(height)
        for (let i = 0; i < list.length; i++) {
          let item = list[i]
          height += item.clientHeight  // 计算每一个group所在的高度区域，从0开始
          this.listHeight.push(height)
        }
      }
    },
    data () {
      return {
        // 监控y轴的位置
        scrollY: -1,
        // 当前索引
        currentIndex: 0,
        // 当从下网上滚动的时候，计算到即将到达滚动容器顶部的元素之间的距离
        diff: -1
      }
    },
    watch: {
      data () {
        setTimeout(() => {
          this._calculateHeight()
        }, 20)
      },
      // 根据滚动的高度计算当前滚动到的索引，然后利用索引就能得到右侧的索引元素
      scrollY (newY) {
        // 滚动是从起始点顶点开始，而触摸滚动是从第一次触点算做起始点
        if (newY >= 0) {
          this.currentIndex = 0
          return
        }
        // 中部区域滚动
        let listHeight = this.listHeight
        for (let i = 0; i < listHeight.length - 1; i++) {
          let height1 = listHeight[i]
          let height2 = listHeight[i + 1]
          let y = -newY // 滚动的话，向下滚动都是负数，从0开始
          // 1. height2 有可能下标越界，获得undefined,所以要么这里判定，要么循环次数减少1
          // 2. 在一个区间内
          if (!height2 || (y > height1 && y < height2)) {
            this.currentIndex = i

            // 计算当前滚动的区域的下一个group的dom元素顶部 距离滚动容器顶部的距离
            // 用来计算 过度效果的偏移像素
            this.diff = height2 + newY
            console.log('diff:', this.diff)
            return
          }
        }
        // 尾部区域
        this.currentIndex = 0
      },
      diff (newDiff) {
        let fixedTop = (newDiff > 0 && newDiff < TITLE_HEIGHT) ? newDiff - TITLE_HEIGHT : 0

        console.log('fixedTop:', fixedTop, 'newDiff:', newDiff)
        // 当不在 TITLE_HEIGHT 范围内的时候，fixedTop 始终返回0.下面再把固定标题框移动到0的位置
        // 且不再继续改变，只有在碰撞范围内，再继续碰撞的效果
        if (this.fixedTop === fixedTop) {
          return
        }
        console.log('---fixedTop:', fixedTop, 'newDiff:', newDiff)
        this.fixedTop = fixedTop
        this.$refs.fixed.style.transform = `translate3d(0,${fixedTop}px,0)`
      }
    }
  }
</script>


<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .list-view {
    position relative // 为了限制住里面的一些元素的绝对定位只能在自己内部
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
    .list-fixed {
      position: absolute;
      top: 0;
      right: 0;
      left: 0;
      .fixed-title {
        font-size: 12px;
        line-height: 30px;
        padding-left: 20px;
        color: rgba(255, 255, 255, 0.5);
        background: #333;
      }
    }
    .loading-container {
      position: absolute
      width: 100%
      top: 50%
      transform: translateY(-50%)
    }
  }
</style>

```
  
      
  
  
  
  