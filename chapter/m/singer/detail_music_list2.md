# 歌手详情页-music-list组件2

前面章节把基础的交互滚动做到了，但是出现了问题，在顶部的时候，由于我们要做的是留出一部分距离，但是这部分距离由于css 和 我们实现的思路原因，出现了问题。下面来讲解怎么解决这个问题

解决问题的思路：

1. 查看dom结构和css的实现，找到当改变背景图部分的z-index则会把列表盖住
2. 那么，当滚动到不再滚动的距离时，改变 背景图部分的z-index和背景图部分的高度
3. 当可以滚动的时候，再重置回去

总结：动画效果完全依赖css和js不断的改变

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

## 下拉让图片有放大缩小的交互效果和上拉高斯模糊
![](/assets/musicapp/歌手详情歌曲列表交互效果图上拉ios高斯模糊.png)
```javascript
watch: {
      scrollY (newY) {
        let translateY = Math.max(this.minTranslateY, newY)
        let zindex = 0
        if (translateY === this.minTranslateY) {
          let bgImageStyle = this.$refs.bgImage.style
          bgImageStyle['padding-top'] = 0
          bgImageStyle['height'] = `${RESERVED_HEIGHT}px`
          zindex = 10
          console.log(`  --- translateY:`, translateY, `newY:`, newY)
        } else {
          // 让背景层跟着网上移动
          this.$refs.bgLayer.style['transform'] = `translate3d(0,${translateY}px,0)`
          let bgImageStyle = this.$refs.bgImage.style
          bgImageStyle['padding-top'] = `70%`
          bgImageStyle['height'] = 0
          zindex = 0
          console.log(`translateY:`, translateY, `newY:`, newY, 'this.minTranslateY:', this.minTranslateY)
        }

        let percent = Math.abs(newY / this.bgImageHeight)
        // 通过缩放来控制图片的大小
        let scale = 1
        let blur = 0
        if (newY > 0) {
          scale = 1 + percent
          zindex = 10  // 解决在下拉的时候，图片虽然有放大缩小的效果，但是由于后来居上，覆盖了放大的显示部分
        } else {
          // 往上拉的时候，不能使用缩放了，因为图片可以放大，但是缩小的话，背景肯定铺不满容器了，很丑陋
          // 那么实现一个高斯模糊的效果，越到顶部越模糊
          blur = Math.min(20 * percent, 20) // 最大限制到20
        }
        // backdrop-filter 可能只有ios才能看到的高斯模糊效果
        this.$refs.filter.style['backdrop-filter'] = `blur(${blur}px)`
        this.$refs.filter.style['webkitBackdrop-filter'] = `blur(${blur}px)`
        let bgImageStyle = this.$refs.bgImage.style
        bgImageStyle['z-index'] = zindex
        bgImageStyle['transform'] = `scale(${scale})`
        bgImageStyle['webkitTransform'] = `scale(${scale})`
      }
    }
```