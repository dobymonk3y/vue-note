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