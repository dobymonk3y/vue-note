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
