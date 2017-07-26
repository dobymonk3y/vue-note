# 环境搭建
使用vue-cli为基础，安装好后，再修改配置文件

## 本地预览环境搭建
本地预览环境用于开发组件的时候 实时预览和调试；
目录如下：
```bash
|- root                     # 项目根目录(项目名)
    |- examples             # 用于本地预览开发
        |- assets
        |- components
        |- router
        |- App.vue
        |- index.html
        |- main.js
    |- src                  # 用于组件开发   
```
* examples目录： 看上面的目录结构，其实就是把原先在src下生成的目录拷贝过来了，我们的思路就是 换了一个文件夹而已。用来写测试开发。

* src目录：用来开发通用组件的代码

### 修改配置文件
由于我们只是把文件目录更换了，尝试模仿原生的配置来写一下；
**config/index.js**
```javascript
# 直接拷贝dev的代码
module.exports = {
    build:{...},  // vue-cli默认配置
    dev:{...},  // vue-cli默认配置
    // 我们模仿dev来写一套配置，直接copydev的配置过来
    examplesDev:{
        env: require('./examples.dev.env'),   // 这里环境变量配置更改为我们新增的
        port: 8080,
        autoOpenBrowser: true,
        assetsSubDirectory: 'static',
        assetsPublicPath: '/',
        proxyTable: {},
        // CSS Sourcemaps off by default because relative paths are "buggy"
        // with this option, according to the CSS-Loader README
        // (https://github.com/webpack/css-loader#sourcemaps)
        // In our experience, they generally work as expected,
        // just be aware of this issue when enabling this option.
        cssSourceMap: false
    } 
}
```

**config/examples.dev.env.js**
```javascript
module.exports = {
  NODE_ENV: '"examples"'
}

```

最简单的配置我们模仿好了，接下来从运行命令入口进行修改
package.json中配置有启动命令的入口文件,如下
```
 "scripts": {
    "dev": "node build/dev-server.js",
    // 我们也需要模仿上面的定义一个文件
    "dev:examples": "node build/dev-examples-server.js",
```

**build/dev-examples-server.js**

同样的，由于我们是增加了一个目录，相当于更改了配置文件的指向路径，我们把原来的build/dev-server.js复制一份进行修改
```javascript
require('./check-versions')()

var config = require('../config')
let nodeenv = process.env.NODE_ENV
if (!nodeenv) {
  // 这里需要修改成我们自己定义好的环境变量
  process.env.NODE_ENV = JSON.parse(config.examplesDev.env.NODE_ENV)
}

var opn = require('opn')
var path = require('path')
var express = require('express')
var webpack = require('webpack')
var proxyMiddleware = require('http-proxy-middleware')

// 这里需要重新按照新增的环境变量判定下，然后赋值
// 为了和验证我们一开始的想法，我们只是修改了影响的部分代码，而不是直接删除以前的代码
var webpackConfig = ''
var customConfig = null
switch (nodeenv) {
  case 'development':
    webpackConfig = require('./webpack.dev.conf')
    customConfig = config.dev
    break
  case 'production':
    webpackConfig = require('./webpack.prod.conf')
    customConfig = config.build
    break
  default:
    // 这里又用到了另外一个配置文件，同样我们需要复制一份./webpack.dev.conf文件进行修改
    webpackConfig = require('./webpack.examples.dev.conf')
    customConfig = config.examplesDev
}

// 下面用到的端口代理配置什么的 都要改成新的配置对象
// default port where dev server listens for incoming traffic
var port = process.env.PORT || customConfig.port
// automatically open browser, if not set will be false
var autoOpenBrowser = !!customConfig.autoOpenBrowser
// Define HTTP proxies to your custom API backend
// https://github.com/chimurai/http-proxy-middleware
var proxyTable = customConfig.proxyTable

var app = express()
var compiler = webpack(webpackConfig)

var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})

var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: () => {},
  heartbeat: 2000
})
// force page reload when html-webpack-plugin template changes
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
    hotMiddleware.publish({action: 'reload'})
    cb()
  })
})

// proxy api requests
Object.keys(proxyTable).forEach(function (context) {
  var options = proxyTable[context]
  if (typeof options === 'string') {
    options = {target: options}
  }
  app.use(proxyMiddleware(options.filter || context, options))
})

// handle fallback for HTML5 history API
app.use(require('connect-history-api-fallback')())

// serve webpack bundle output
app.use(devMiddleware)

// enable hot-reload and state-preserving
// compilation error display
app.use(hotMiddleware)

// serve pure static assets
var staticPath = path.posix.join(customConfig.assetsPublicPath, customConfig.assetsSubDirectory)
app.use(staticPath, express.static('./static'))

var uri = 'http://localhost:' + port

var _resolve
var readyPromise = new Promise(resolve => {
  _resolve = resolve
})

console.log('> Starting dev server...')
devMiddleware.waitUntilValid(() => {
  console.log('> Listening at ' + uri + '\n')
  // when env is testing, don't need open it
  if (autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    opn(uri)
  }
  _resolve()
})

var server = app.listen(port)

module.exports = {
  ready: readyPromise,
  close: () => {
    server.close()
  }
}

```

**build/webpack.examples.dev.conf.js**

```javascript
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
var path = require('path');

// 原生的配置文件只考虑了 打包成web应用的开发和发布，他们是同一个目录
// 所以这里我要定义 不同的目录，就要先覆盖baseWebpackConfig里面的入口文件和输出目录
let entry = {
  app: './examples/main.js'
}
// add hot-reload related code to entry chunks
// 这里是热加载的配置
// 下面的代码作用完成之后把入口文件修改成了一个数组 app: [ './build/dev-client', './examples/main.js' ]
Object.keys(entry).forEach(function (name) {
  entry[name] = ['./build/dev-client'].concat(entry[name])
})

module.exports = merge(baseWebpackConfig, {
  entry: entry,
  // 输出
  // 覆盖基础配置里面的输出，把输出的目录定义到 examples/dist目录下
  output: {
    path: path.join(__dirname, '../examples/dist'),
    publicPath: '',
    filename: '[name].js',
    chunkFilename: '[name].chunk.js'
  },
  module: {
    rules: utils.styleLoaders({sourceMap: config.dev.cssSourceMap})
  },
  // cheap-module-eval-source-map is faster for development
  devtool: '#cheap-module-eval-source-map',
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.examplesDev.env
    }),
    // https://github.com/glenjamin/webpack-hot-middleware#installation--usage
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoEmitOnErrorsPlugin(),
    // https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({
      filename: path.join(__dirname, '../examples/dist/index.html'),
      template: path.join(__dirname, '../examples/index.html'),
      inject: true
    }),
    new FriendlyErrorsPlugin()
  ]
})
```