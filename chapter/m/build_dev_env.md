# 环境搭建

## vue-cli 初始化

```bash
$ vue init webpack vue-music

? Project name music
? Project description 音乐播放器
? Author zhuqiang <zhuqiang@xxx.net>
? Vue build runtime                           // standalone 独立编译，runtime运行时编译，百度吧，没太看明白说的是啥意思，对于普通用户来说应该只是文件变小了而已
                      //standalone：Runtime + Compiler；runtime：Runtime-only 两种模式选择
? Install vue-router? Yes                      // 使用路由
? Use ESLint to lint your code? Yes            // 使用js代码规范
? Pick an ESLint preset Standard               // 并选择这个标准规范,可以用上下方向键选择
? Setup unit tests with Karma + Mocha? No      // 不需要测试
? Setup e2e tests with Nightwatch? No

   vue-cli · Generated "vue-music".

   To get started:

     cd vue-music
     npm install
     npm run dev

   Documentation can be found at https://vuejs-templates.github.io/webpack
```

初始化完成之后，按照上面的提示，进行install和run dev 就能看到浏览器中的页面了。这里不得不说的是：在360急速浏览器中使用 IE11模式，打开该项目白屏，改成急速模式的话正常，而且可以在ios原生浏览器中打开。（以前做的项目不知道是什么原因，不能在ios中打开）

### standalone 和 runtime 模式
这是 runtime  模式下生成的vue初始化代码。
```javascript
new Vue({
  el: '#app',
  router,
  render: h => h(App)
})

// 上面的 render: h => h(App) 中的h是createElement函数，是es6的简写语法
```

这是 standalone 模式下生成的代码
```javascript
new Vue({
  el: '#app',
  router,
  store,
  template: '<App/>',
  components: {App}
})

```

可以看出来。两种模式下的挂载方式不一样，runtime 使用 render。而 standalone 使用 template加components；
经过测试：这两种模式对于白屏问题 都不受该猜想的影响。
