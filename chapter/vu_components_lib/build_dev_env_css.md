# 环境搭建-单独css打包配置

这个打包，是针对一份css的文件(或则是预处理的文件)单独进行编译打包成最终的css文件。所以说这个功能应该是比较独立的。 这个直接使用iview中css打包脚本。

但是我们还是按照之前的配置 统一配置基础配置的套路进行一步一步的添加。就是想要有一个打包后的清单.哈哈哈..

**config/index.js**

老规矩还是先增加一个环境配置，再配置中有需要提取出来的再增加具体的配置
```jacascript
module.exports = {
    ...
   buildStyl:{
    env: require('./prod.styl.env'),
  }
}
```

**package.json**

