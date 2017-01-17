# 使用vue2搭建开发环境
## 初始化项目脚手架
### vue-lic 安装

  参考官网：https://github.com/vuejs/vue-cli
### 查看vue-cli 支持有
```bash
$ vue list

  Available official templates:

  ★  browserify - A full-featured Browserify + vueify setup with hot-reload, linting & unit testing.
  ★  browserify-simple - A simple Browserify + vueify setup for quick prototyping.
  ★  simple - The simplest possible Vue setup in a single HTML file
  ★  webpack - A full-featured Webpack + vue-loader setup with hot reload, linting, testing & css extraction.
  ★  webpack-simple - A simple Webpack + vue-loader setup for quick prototyping.
```

### 使用webpack初始化项目

```bash
$ vue init webpack vue-webpack-demo

  This will install Vue 2.x version of the template.

  For Vue 1.x use: vue init webpack#1.0 vue-webpack-demo2

? Project name vue-webpack-demo
? Project description A Vue.js project
? Author zhuqiang <zhuqiang@tidebuy.net>
? Vue build standalone
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Setup unit tests with Karma + Mocha? No
? Setup e2e tests with Nightwatch? No

   vue-cli · Generated "vue-webpack-demo2".

   To get started:

     cd vue-webpack-demo2
     npm install
     npm run dev

   Documentation can be found at https://vuejs-templates.github.io/webpack
```
说明：
```bash
? Project name vue-webpack-demo        
? Project description A Vue.js project 
? Author zhuqiang <zhuqiang@tidebuy.net>
? Vue build standalone  
? Use ESLint to lint your code? Yes //是应用eslint语法检查，强制根据配置规则进行检查语法
? Pick an ESLint preset Standard // 直接回车

```
