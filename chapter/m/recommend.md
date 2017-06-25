# 推荐页面 + jsonp获取数据

## 数据接口调研
在谷歌浏览器中切换到手机模式，访问 `https://m.y.qq.com/`，然后查看请求，找到需要的请求链接，因为是抓去别人的数据，所以人家获取数据的方式不是一层不变的。 请求过滤：
1. xhr 是ajax请求
2. js 是获取的一些js文件，也包括跨域数据，这就要说道jsop原理了

## jsonp
https://github.com/webmodules/jsonp