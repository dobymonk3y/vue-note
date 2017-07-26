# 环境搭建-组件打包环境

组件打包环境 目前我的理解是，把要独立出来的组件 全部放到了一个固定的目录，和 开发测试环境分开了。

组件打包的目录如下：
```bash
| - rootxxx                 # 项目根目录（项目名）
    |- src                  # 用于组件开发,vue-li默认生成的目录
        |- components       # 我们所有的组件都放在这个里面 
          |- hello          # hello组件目录
            |-Hello.vue     
            |-index.js      # 模块化暴露文件
```

本次只配置组件打包和使用，其他的需求以后再慢慢深入了解

## 编写第一个组件
**src/components/hello/Hello.vue**
```javascript
<template>
  <div class="hello">
    <h1>{{ title }}</h1>
    <h3>{{ msg }}</h3>
    <ul>
      <li><a href="http://router.vuejs.org/" target="_blank">vue-router</a></li>
      <li><a href="http://vuex.vuejs.org/" target="_blank">vuex</a></li>
      <li><a href="http://vue-loader.vuejs.org/" target="_blank">vue-loader</a></li>
      <li><a href="https://github.com/vuejs/awesome-vue" target="_blank">awesome-vue</a></li>
    </ul>
  </div>
</template>

<script>
  export default {
    props: {
      title: String
    },
    data () {
      return {
        msg: 'hello word'
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
    background-color: #9ebdd7;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }
</style>

```
**src/components/hello/index.js**
```javascript
import Hello from './Hello'
export default Hello
```

到此为止我们的组件写完了，测试怎么测试呢？

* 直接和平时开发webapp一样，可以直接引入该vue文件。
* 还可以使用 Vue.use(xx) 安装的方式来引入文件，这种方式下面讲

## 实现Vue.use全局安装我们的组件
由于我们写的是组件库。所以要暴露很多不同的组件。需要一个统一的js来进行安装。

**src/index.js**

```javascript
/**
 *
 * <pre>
 *  Version         Date            Author          Description
 * ---------------------------------------------------------------------------------------
 *  1.0.0           2017/07/25     zhuqiang        -
 * </pre>
 * @author zhuqiang
 * @version 1.0.0 2017/7/26 13:54
 * @date 2017/7/26 13:54
 * @since 1.0.0
 */
// 这个难道是一个es6的补丁？正常的vue-cli中已经包含了补丁了应该。所以这个暂时不知道去掉后怎么重现故障
import 'core-js/fn/array/find-index'

import Hello from './components/hello'
let VueComponentsLib = {
  Hello
}

const install = function (Vue, opts = {}) {
  Object.keys(VueComponentsLib).forEach((key) => {
    Vue.component(key, VueComponentsLib[key])
  })
}

// auto install
if (typeof window !== 'undefined' && window.Vue) {
  install(window.Vue)
}
// 把 install方法添加到 VueComponentsLib 中
export default Object.assign(VueComponentsLib, {install})
// module.exports = Object.assign(tlzVueElUi, {install})   // eslint-disable-line no-undef

```

这个测试方法如下：
在examples/main.js中添加如下代码
```javascript
import VueComponentsLib from '../src/index'
Vue.use(VueComponentsLib)
```
好了，这个被全局安装了，至于按需安装的话，我就没有研究了，后台系统全局安装能满足我的需求了。

## 打包成js
我看百度有直接把vue打包出去的。这个方法不知道有用没有，打包成js的话，说得最多的是，在非vue-cli环境下也能使用。或许在vue-cli环境下直接使用这里的 .vue文件的组件也是可以的把？

首先呢，同本地预览的思路差不多，我们先增加基础的配置

**config/index.js**

在以前的基础上增加一个配置环境，不过这里暂时还不知道需要哪些变量配置，因为直接打成js包和webapp包还不一样

```javascript
  buildProdLib:{
    env: require('./prod.lib.env.js'),
  }
```

**config/prod.lib.env.js**
```javascript
module.exports = {
  NODE_ENV: '"prodlib"'
}

```

基础参数配置好了，接下来还是从命令行入口处开始
**package.json**
```javascript
  "scripts": {
    "build": "node build/build.js",
    // 同样模仿上面的默认配置来修改
    "build:prodLib": "node build/buildProdLib.js",
```

**build/buildProdLib.js**
复制了一份build/build.js来修改。
这个文件是启动构建并显示构建后的文件大小。就和原生构建web应用时的效果一样
```javascript
require('./check-versions')()

// 这里的环境改成我们自己的
process.env.NODE_ENV = 'prodlib'

var ora = require('ora')
var rm = require('rimraf')    // 这个查了下，大概是清理文件的插件
var path = require('path')
var chalk = require('chalk')
var webpack = require('webpack')
var config = require('../config')
// 这里同理复制一份 webpack.prod.conf.js 修改成 webpack.prodLib.conf.js
var webpackConfig = require('./webpack.prodLib.conf')

var spinner = ora('building for production...')
spinner.start()

// 也就是说，这里帮我们清理了以下目录
// 然而这里我们只需要清空自己的打包目录就行了
// 之前在config/index.js中没有定义多的变量，在这里要用到了，需要添加上
//  assetsRoot: path.resolve(__dirname, '../lib'),
rm(path.join(config.buildProdLib.assetsRoot), err => {
  if (err) throw err
  webpack(webpackConfig, function (err, stats) {
    spinner.stop()
    if (err) throw err
    process.stdout.write(stats.toString({
      colors: true,
      modules: false,
      children: false,
      chunks: false,
      chunkModules: false
    }) + '\n\n')

    console.log(chalk.cyan('  Build complete.\n'))
    console.log(chalk.yellow(
      '  Tip: 构建的文件是js库的文件\n' +
      '  因此该库能被引用使用\n'
    ))
  })
})

```
**build/webpack.prodLib.conf.js**
```javascript
var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
// 下面这四个插件没有什么用了
var CopyWebpackPlugin = require('copy-webpack-plugin')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')

// 这里如果不是测试环境，就改成我们自己的打包环境
//  暂时这个测试的我还不会，但是看到开源框架都安装了这个unit测试
// 所以这里暂时不用理会其他的。环境变量先改成我们自己的就好
var env = process.env.NODE_ENV === 'testing'
  ? require('../config/test.env')
  : config.buildProdLib.env

// 该模块下由于咱们是打成js包,所以很多文件都不需要生成和处理
// 这里啰嗦一句，从配置本地预览开发环境和打包环境 都是覆盖了baseWebpackConfig中的入口和输出目录
// 所以建议直接把baseWebpackConfig中的entry和output移除
var webpackConfig = merge(baseWebpackConfig, {
  entry: {
    app: './src/index.js'
  },
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  // devtool: config.build.productionSourceMap ? '#source-map' : false,
  output: {
    path: config.buildProdLib.assetsRoot,
    // 再次增加一个变量目录
    publicPath: config.buildProdLib.assetsPublicPath,
    // 库文件名，同样我们需要把这两个都写到配置里面去
    filename: config.buildProdLib.filename,
    // 库名称
    library: config.buildProdLib.library,
    libraryTarget: 'umd',
    umdNamedDefine: true
  },
  // 增加一个配置，在iview中定义的，百度说是支持不同的加载方式
  externals: {
    vue: {
      root: 'Vue',
      commonjs: 'vue',
      commonjs2: 'vue',
      amd: 'vue'
    }
  },
  plugins: [
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
    new webpack.DefinePlugin({
      'process.env': env
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      sourceMap: true
    }),
    // extract css into its own file
    // new ExtractTextPlugin({
    //   filename: utils.assetsPath('css/[name].[contenthash].css')
    // }),
    // Compress extracted CSS. We are using this plugin so that possible
    // duplicated CSS from different components can be deduped.
    // new OptimizeCSSPlugin({
    //   cssProcessorOptions: {
    //     safe: true
    //   }
    // }),
    // 这里的HtmlWebpackPlugin插件用不到了，直接删除掉了
    // 下面的插件也是用不到的了
    // split vendor js into its own file
    // new webpack.optimize.CommonsChunkPlugin({
    //   name: 'vendor',
    //   minChunks: function (module, count) {
    //     // any required modules inside node_modules are extracted to vendor
    //     return (
    //       module.resource &&
    //       /\.js$/.test(module.resource) &&
    //       module.resource.indexOf(
    //         path.join(__dirname, '../node_modules')
    //       ) === 0
    //     )
    //   }
    // }),
    // 这个配置应该是压缩js的
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      sourceMap: false
    }),
// extract webpack runtime and module manifest to its own file in order to
// prevent vendor hash from being updated whenever app bundle is updated
//     new webpack.optimize.CommonsChunkPlugin({
//       name: 'manifest',
//       chunks: ['vendor']
//     }),
// copy custom static assets
//     new CopyWebpackPlugin([
//       {
//         from: path.resolve(__dirname, '../static'),
//         to: config.build.assetsSubDirectory,
//         ignore: ['.*']
//       }
//     ])
  ]
})

// 这个压缩没有什么用了
// if (config.build.productionGzip) {
//   var CompressionWebpackPlugin = require('compression-webpack-plugin')
//
//   webpackConfig.plugins.push(
//     new CompressionWebpackPlugin({
//       asset: '[path].gz[query]',
//       algorithm: 'gzip',
//       test: new RegExp(
//         '\\.(' +
//         config.build.productionGzipExtensions.join('|') +
//         ')$'
//       ),
//       threshold: 10240,
//       minRatio: 0.8
//     })
//   )
// }
// npm run build:prodLib --report  显示构建报表
if (config.buildProdLib.bundleAnalyzerReport) {
  var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}

module.exports = webpackConfig

```
文件就配置完成了，补充下在修改文件配置的时候增加的配置变量
**config/index.js**
打包环境的配置变量如下
```javascript
  buildProdLib: {
    env: require('./prod.lib.env.js'),
    assetsRoot: path.resolve(__dirname, '../lib'),
    assetsPublicPath: '/lib/',
    filename:'vueComponentsLib.min.js',
    library:'vueComponentsLib',
    bundleAnalyzerReport: process.env.npm_config_report
  }
```

## 打包测试
```bash
# 只打包
npm run build:prodLib
# 打包并查看报表
npm run build:prodLib --report
```
构建完成后生成`lib/vueComponentsLib.min.js` 文件

## 使用打包后的js文件进行测试
在开发测试环境测试
**examples/main.js**
```
# 把直接引用的js替换成打包好的js
// import VueComponentsLib from '../src/index'
import VueComponentsLib from '../lib/vueComponentsLib.min.js'
```

最终成功