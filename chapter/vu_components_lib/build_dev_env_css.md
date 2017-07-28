# 环境搭建-单独css打包配置

这个打包，是针对一份css的文件(或则是预处理的文件)单独进行编译打包成最终的css文件。所以说这个功能应该是比较独立的。 这个直接使用iview中css打包脚本。

这个配置是基于gulp的所以，没有办法直接修改之前的配置了。但是我们还是可以把一些变动的配置提取出来

**config/index.js**

老规矩还是先增加一个环境配置，再配置中有需要提取出来的再增加具体的配置
```jacascript
module.exports = {
    ...
   buildStyl:{
  }
}
```
**
本次配置需要添加以下几个依赖项**
```jacascript
  "devDependencies": {
    ...
    "gulp": "^3.9.1",
    "gulp-autoprefixer": "^3.1.1",
    "gulp-clean-css": "^3.7.0",
    "gulp-stylus": "^2.6.0",
    "gulp-rename": "^1.2.2",
    "stylus": "^0.54.5",
    "stylus-loader": "^3.0.1"
  }
```

