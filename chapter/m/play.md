# 播放器组件

![](/assets/musicapp/播放器页面设计详解.png)

如上图，这次要做的播放器效果图如上；大体的功能如下：

1. 第一张图：播放器界面
    1. 模糊背景
    2. 圆形旋转唱片
    3. 当前歌词
    4. 进度条
    5. 播放模式按钮
    6. 上一首、下一首按钮
    7. 播放、暂停按钮
    8. 搜藏（保存到本地）
2. 第二张图片：歌词界面
    
    歌词界面和播放器界面可以互相左右滑动切换。功能如下
    1. 最下边的功能和播放器界面的功能一致
    2. 展示整个歌词，提示当前歌词，并随着歌曲时间滚动歌词
3. 第三张图片：歌手详情歌曲列表界面，点击第一、二张图片的左上角的按钮，显示的本页面
    1. 之前做的歌手详情和列表功能
    2. 最底部有一个迷你的播放条
        1. 展示歌曲的简单信息
        2. 播放、暂停按钮，还有一个圆圈进度条，表示当前歌曲播放的进度
        3. 当前正在播放的歌曲列表按钮
        
## 拆分全局状态共享数据

由于这个播放器组件可以在很多页面都能打开，并控制，所以播放器的数据是共享数据。因此我们要把这些基础数据抽象出来用vuex来存储

** src/store/state.js    抽象要管理的状态数据**
```javascript
import { playMode } from '../common/js/config'

const state = {
  /* 歌手页面跳转到歌手详情页面传递的参数 */
  singer: {},
  /* 播放器播放状态：播放？暂停 */
  playing: false,
  /* 全屏模式 ？ 迷你模式 */
  fullScreen: false,
  /* 原始播放列表数据 */
  playlist: [],
  /* 顺序列表:支持2种：1. 顺序播放(和playlist顺序一致)，2. 随机播放  */
  sequenceList: [],
  /* 播放模式：支持三种：1. 随机 2.顺序 3.单曲循环  */
  mode: playMode.sequence,
  /* 当前正常播放的歌曲索引 */
  currentIndex: -1
}

export default state
```    

 由于上面播放模式是固定的几个常量，所以我们把它提取出来放到一个配置文件中
 
** src/common/js/config.js**
```javascript
/** ====================================
 *  我们把项目中一些配置放到一个文件中
 * ====================================
 */

/**
 *  播放模式
 *  1.  顺序
 *  2. 循环
 *  3. 随机
 * @type {{sequence: number, loop: number, random: number}}
 */
export const playMode = {
  sequence: 0,
  loop: 1,
  random: 2
}
```   

状态数据有了。接下来我们要写怎么获取数据，也就是 getters

getters中对获取数据做一个简单的封装（也可以做一些复杂的计算），也相当于一个计算属性

**src/store/getters.js**
```javascript
/** 用于获取数据 */

export const singer = state => state.singer

export const playing = state => state.playing
export const fullScreen = state => state.fullScreen
export const playlist = state => state.playlist
export const sequenceList = state => state.sequenceList
export const mode = state => state.mode
export const currentIndex = state => state.currentIndex

```    

能获取数据了，那么怎么修改数据呢？ 编写 mutations

在编写mutations之前，为了方便获取和操作方法，把方法名定义常量type

**src/store/mutations-types.js**
```javascript
export const SET_SINGER = 'SET_SINGER'

export const SET_PLAYING = 'SET_PLAYING'
export const SET_FULL_SCREEN = 'SET_FULL_SCREEN'
export const SET_PLAYLIST = 'SET_PLAYLIST'
export const SET_SEQUENCE_LIST = 'SET_SEQUENCE_LIST'
export const SET_MODE = 'SET_MODE'
export const SET_CURRENT_INDEX = 'SET_CURRENT_INDEX'
```

在mutations中可以提交非异步的数据。
**src/store/mutations.js**
```javascript
import * as types from './mutations-types'

const mutations = {
  [types.SET_SINGER] (state, singer) {
    state.singer = singer
  },
  [types.SET_PLAYING] (state, playing) {
    state.playing = playing
  },
  [types.SET_FULL_SCREEN] (state, fullScreen) {
    state.fullScreen = fullScreen
  },
  [types.SET_PLAYLIST] (state, playlist) {
    state.playlist = playlist
  },
  [types.SET_SEQUENCE_LIST] (state, sequenceList) {
    state.sequenceList = sequenceList
  },
  [types.SET_MODE] (state, mode) {
    state.mode = mode
  },
  [types.SET_CURRENT_INDEX] (state, currentIndex) {
    state.currentIndex = currentIndex
  }
}
```

在action中可以封装对多个mutations的提交，也可以提交异步数据，但是这里暂时还用不上，等待编写细节的时候如有用到再添加。
**src/store/action.js**
```javascript
暂无数据
```

## 初版播放器应用架子

问题：

1. 先做一个播放器的初始骨架组件
2. 这个骨架组件在哪里引用？才能被多个入口控制呢？
  
  由于该组件被多个入口控制，就说明这个也是一个全局的，那么放在App.vue中，其他入口控制 根据改变vuex中的数据来控制播放器的行为。

> ## 从这一张开始，先看完视频然后再靠着记忆和理解，完成本章节所展示的内容。包括css。 下一章节肯定会对这一章节进行大量的修改，这些都不要紧。有思路就好


由于这个是一个业务组件，先建立初始的框架页面
**src/components/player/player.vue   **

初始架子所要做的功能就是：

1. 有播放列表的时候才显示播放组件
2. 此时播放器是全屏还是迷你 


```html
<template>
  <!--主要分为两大块内容：1. 全屏播放器 2， 迷你播放器-->
  <div class="player" v-show="playlist.length >0">
    <div class="normal-player" v-show="fullScreen">
      播放器
    </div>
    <div class="mini-player" v-show="!fullScreen">
      迷你播放器
    </div>
  </div>
</template>
```
```javascript
<script type="text/ecmascript-6">
  import { mapGetters } from 'vuex'

  export default {
    data () {
      return {}
    },
    computed: {
      ...mapGetters([
        'fullScreen',
        'playlist'
      ])
    }
  }
</script>
```
```css
<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"
  @import "~common/stylus/mixin"

  .player {
    position: absolute;
    top: 0;
    right: 0;
    left: 0;
    bottom: 0;
    background $color-background
    .mini-player {
      position absolute
      bottom 0
      right 0
      left 0
      height 60px
      background $color-highlight-background
    }
  }
</style>
```     

在App中引用
**src/App.vue**
```javascript
<template>
  <div id="app">
    <m-header></m-header>
    <tab></tab>
    <keep-alive>
      <router-view></router-view>
    </keep-alive>
    <player></player>
  </div>
</template>

<script>
  // 组件首字母大写，因为组件以css命名的
  import MHeader from 'components/m-header/m-header'
  import Tab from 'components/tab/tab'

  import Player from 'components/player/player'

  export default {
    name: 'app',
    components: {MHeader, Tab, Player},
    data () {
      return {}
    }
  }
</script>
```  
这个时候，由于播放列表没有数据，所以页面不会有任何的变化。接下来，接着上一个大章节的，歌手详情歌曲列表中点击一首歌曲，然后打开这个播放器页面。

  
这里需要注意的是：之前的歌曲列表组件里面 引用了一个 songList组件，这个组件里面只做最基础的列表渲染，不做任何业务，因为被多个地方引用？
所以需要先对该该组件进行功能增加。

1. src/base/song-list/song-list.vue 派发点击事件

  点击的是列表中的一首歌曲列表，所以需要回调以下参数
  1. 当前歌曲信息
  2. 当前歌曲在列表中的索引
2. src/components/music-list/music-list.vue 中监听点击事件，并更改vuex中的相关状态。
  
  这里需要更改多个状态才行
  1. 播放列表
  2. 原始数据列表（因为我们有播放模式）
  2. 当前歌曲的索引
  3. 播放器全屏还是迷你模式（在这个场景中，点击歌曲就应该是播放了。跳转到播放列表）
  4. 播放状态
  
简单的就不记录了，以后只记录关键的思路

由于这里第二步骤，控制播放器而修改状态的时候要提交多个mutations，所以这里使用action来封装这个操作

**src/store/action.js**
```javascript
import * as types from './mutations-types'

// 官方语法 https://vuex.vuejs.org/zh-cn/actions.html
export const selectPlay = function ({commit, state}, {list, index}) {
  // 先暂时播放列表和排序列表一致
  // 且 打开全屏播放器模式，并且设置为播放状态
  commit(types.SET_SEQUENCE_LIST, list)
  commit(types.SET_PLAYLIST, list)
  commit(types.SET_CURRENT_INDEX, index)
  commit(types.SET_FULL_SCREEN, true)
  commit(types.SET_PLAYING, true)
}
```

在music-list中使用action提交

**src/components/music-list/music-list.vue**


```javascript
// 使用语法糖

  // node_modules/vuex/types/helpers.d.ts 在这个文件中 有语法糖的名称
  import { mapActions } from 'vuex'
  export default {
      methods: {
      ...mapActions([
        'selectPlay'
      ]),
      selectItem (item, index) {
      // 这里提交的时候 一定要提交对象，因为只能提交一个参数
        this.selectPlay({
          list: this.songs,
          index
        })
      }
  }
```

  
        

            