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

getters中对获取数据做一个简单的封装，也相当于一个计算属性

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


            