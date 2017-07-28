# 环境搭建-npm上传配置教程
npm 上传的请移步这里:https://zq99299.gitbooks.io/gitbook-guide/content/chapter/node_npm.html#插件发布到npm社区

**package.json**
package.json 中我们需要关注的应该是以下几个属性

- version : 1.0.0        # 版本号，每次发布不能重复
- dependencies ：{}      # 依赖的库，vue-cli默认安装的因为我们需要调试所以安装的时候使用了 vue-router，但是打包的时候不需要，所以这个应该移动到devDependencies中去，只是开发的时候测试使用   

**.npmignore**
决定哪些文件被忽略不发布到npm中去。
![](/assets/image/参考饿了吗ui把其他的不发布.png)