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

**本次配置需要添加以下几个依赖项**

```jacascript
  "devDependencies": {
    ...
    "gulp": "^3.9.1",
    "gulp-autoprefixer": "^3.1.1",
    "gulp-clean-css": "^3.7.0",
    "gulp-rename": "^1.2.2",
    // 由于我们这次采用stylus来作为css的预处理器，所以要增加stylus相关的预处理器
    "gulp-stylus": "^2.6.0",
    // 这里的是之前没有说道的，用于测试预览环境中vue文件中可以写stylus代码需要安装的处理器
    "stylus": "^0.54.5",
    "stylus-loader": "^3.0.1"
  }
```

**package.json**
增加命令入口
```jacascript
  "scripts": {
    ...
   "build:prodStyl": "gulp --gulpfile build/buildProdStyl.js",
```

**build/buildProdStyl.js**
编写打包的配置文件
```jacascript
var gulp = require('gulp')
var cleanCSS = require('gulp-clean-css')
var stylus = require('gulp-stylus')
var rename = require('gulp-rename')
var autoprefixer = require('gulp-autoprefixer')
var rm = require('rimraf')
var config = require('../config')

// 先删除目录
gulp.task('clean', function () {
  rm(config.buildStyl.output, err => {
    if (err) throw err
  })
})

// 编译
gulp.task('css', function () {
  // 把要编译的stylus文件路径作为变量配置到环境配置中
  gulp.src(config.buildStyl.source)
    .pipe(stylus())
    .pipe(autoprefixer({
      browsers: ['last 2 versions', 'ie > 8']
    }))
    .pipe(cleanCSS())
    .pipe(rename(config.buildStyl.filename))
    .pipe(gulp.dest(config.buildStyl.output))
})

// 拷贝字体文件
gulp.task('fonts', function () {
  gulp.src(config.buildStyl.sourceFonts)
    .pipe(gulp.dest(config.buildStyl.outputFonts))
})

gulp.task('default', ['clean','css', 'fonts'])


```

最后补充下我们的环境变量配置增加的变量完整如下;
**config/index.js**

```jacascript
module.exports = {
    ...
     buildStyl: {
      // 要打包的源文件
      source: path.resolve(__dirname, '../src/styl/index.styl'),
      // 字体文件目录
      sourceFonts:path.resolve(__dirname, '../src/styl/fonts/*.*'),
      // 输出目录
      output: path.resolve(__dirname, '../lib/styl/'),
      outputFonts: path.resolve(__dirname, '../lib/styl/fonts'),
      // 输出文件名称
      filename: 'vueComponentsLib.min.css'
    }
}
```



