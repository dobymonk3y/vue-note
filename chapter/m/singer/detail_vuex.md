# 歌手详情页 - vuex

上一节中讲到通过path传递id，但是这样只能传递简单的参数，如果要传递复杂的参数怎么办呢？ 那么就要用到vuex

官方教程有基础的讲解。下面就来看看怎么做的

**先看下store下的文件目录结构：**
可以看出来基本上把官方的简单例子中的结构都分离到各个文件中了
```bash
|- store        # vuex根目录
    |- action.js     # 
    |- getters.js    # 用于获取数据，而不是直接从state获取
    |- mutations.js  # 提交数据操作
    |- mutations-types.js # 常量字符串
    |- state.js           # 数据   
```
**src/store/state.js 定义数据状态有哪些**
```javascript
const state = {
  singer: {}
}

export default state
```

**src/store/mutations.js 定义提交修改的方法**
```javascript
/** 用于提交 */

import * as types from './mutations-types'

const mutations = {
  [types.SET_SINGER] (state, singer) {
    state.singer = singer
  }
}

export default mutations
```

**src/store/mutations-types.js 把方法名提取出来，方便书写和引用**
```javascript
export const SET_SINGER = 'SET_SINGER'
```

**src/store/getters.js 定义获取数据的方法**
```javascript
/** 用于获取数据 */

// es6 箭头表达式缩写
export const singer = state => state.singer
```
```javascript
// 上面的简写相当于这个下面这个
export function singer2 (state) {
return state.singer
}
```

**src/store/action.js ： 暂时用不上**

**src/store/index.js 最后把分散的各个部分组装起来**

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

import * as actions from './action'
import * as getters from './getters'
import mutations from './mutations'
import state from './state'

// 开发模式下使用日志，和谷歌浏览器里面的vuedevtool插件类似
// 子修改state的时候会在控制台打印一些信息
import createLogger from 'vuex/dist/logger'

Vue.use(Vuex)

// 调试,开发模式开启严格模式，要使用mutations提交
const debug = process.env.NODE_ENV !== 'production'

export default new Vuex.Store({
  actions,
  getters,
  state,
  mutations,
  strict: debug,
  plugins: debug ? [createLogger()] : []
})

```

注意：上面的文件中用到了 import * as actions的语法。

**import * as xxx 和 import xxx 的区别是什么？**
```javascript
import * as xxx 和 import {xx,x2} : 不能用于export default xxx 导出的js文件，* as xxx 是表示把里面的都定义到一个容器 xxx下。而后面解构的变量名必须和文件中导出的一致
import xxx ：只能用于 export default xxx 导出的js文件，且xxx可以和导出的变量名不一致。


比如下面的
const xx = {x1,x2}
export default xx

import * as xx : 导出的数据结构是 {default:{x1,x2}}
import xxx ：导出的数据结构是{x1,x2}


```



