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
config/index.js
```javascript
# 直接拷贝dev的代码
module.exports = {
    build:{...},  // vue-cli默认配置
    dev:{...},  // vue-cli默认配置
    examplesDev:{..}  // 我们模仿dev来写一套配置，直接copydev的配置过来
}
```


