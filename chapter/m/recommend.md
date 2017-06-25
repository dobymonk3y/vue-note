# 推荐页面 + jsonp获取数据

## 数据接口调研
在谷歌浏览器中切换到手机模式，访问 `https://m.y.qq.com/`，然后查看请求，找到需要的请求链接，因为是抓去别人的数据，所以人家获取数据的方式不是一层不变的。 请求过滤：
1. xhr 是ajax请求
2. js 是获取的一些js文件，也包括跨域数据，这就要说道jsop原理了

## jsonp
https://github.com/webmodules/jsonp

> 很简单，就是利用`<script>`标签没有跨域限制的“漏洞”（历史遗迹啊）来达到与第三方通讯的目的。
 当需要通讯时，本站脚本创建一个`<script>`元素，地址指向第三方的API网址，形如： `<script src="http://www.example.net/api?param1=1&param2=2"></script>`     并提供一个回调函数来接收数据（函数名可约定，或通过地址参数传递）。     
 第三方产生的响应为json数据的包装（故称之为jsonp，即json padding），形如：     `callback({"name":"hax","gender":"Male"})`     这样浏览器会调用`callback`函数，并传递解析后json对象作为参数。本站脚本可在`callback`函数里处理所传入的数据。    
补充：“历史遗迹”的意思就是，如果在今天重新设计的话，也许就不会允许这样简单的跨域了嘿，比如可能像XHR一样按照CORS规范要求服务器发送特定的http头。