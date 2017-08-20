# 歌手详情页-music-list组件2

前面章节把基础的交互滚动做到了，但是出现了问题，在顶部的时候，由于我们要做的是留出一部分距离，但是这部分距离由于css 和 我们实现的思路原因，出现了问题。下面来讲解怎么解决这个问题

解决问题的思路：

1. 查看dom结构和css的实现，找到当改变背景图部分的z-index则会把列表盖住
2. 那么，当滚动到不再滚动的距离时，改变 背景图部分的z-index和背景图部分的高度
3. 当可以滚动的时候，再重置回去

```javascript
scrollY (newY) {
        let translateY = Math.max(this.minTranslateY, newY)
        if (translateY === this.minTranslateY) {
//          let zindex = 0
          let bgImageStyle = this.$refs.bgImage.style
          bgImageStyle['padding-top'] = 0
          bgImageStyle['height'] = `${RESERVED_HEIGHT}px`
          bgImageStyle['z-index'] = 10
          console.log(`  --- translateY:`, translateY, `newY:`, newY)
        } else {
          // 让背景层跟着网上移动
          this.$refs.bgLayer.style['transform'] = `translate3d(0,${translateY}px,0)`
          let bgImageStyle = this.$refs.bgImage.style
          bgImageStyle['padding-top'] = `70%`
          bgImageStyle['height'] = 0
          bgImageStyle['z-index'] = 0
          console.log(`translateY:`, translateY, `newY:`, newY)
        }
```