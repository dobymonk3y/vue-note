# 推荐页面 + jsonp获取数据

## 数据接口调研
在谷歌浏览器中切换到手机模式，访问 `https://m.y.qq.com/`，然后查看请求，找到需要的请求链接，因为是抓去别人的数据，所以人家获取数据的方式不是一层不变的。 请求过滤：
1. xhr 是ajax请求
2. js 是获取的一些js文件，也包括跨域数据，这就要说道jsop原理了

这次获取到的是该地址返回的页面所需要的数据：`https://c.y.qq.com/musichall/fcgi-bin/fcg_yqqhomepagerecommend.fcg?g_tk=5381&uin=0&format=json&inCharset=utf-8&outCharset=utf-8&notice=0&platform=h5&needNewCode=1&_=1498359279556`

视频中是jsonp访问的。而我看的时候获取到的是同源访问，那么联想到视频中抓去到请求参数`jsonpCallback=jsonp1`,尝试加了该参数，访问的数据就是以json1{xxx} 的形式返回的了。很厉害，一个接口支持所有的格式返回


注意：在抓包获取接口的时候，有可能会遇到特殊处理的比如；pc站的qq音乐的歌单列表
```
https://y.qq.com/portal/playlist.html#t2=5&
上面是站点页面，下面是歌单列表地址，直接请求都显示无法显示网页，但是刷新页面能看到。这样的就是做了一些特殊处理的。
https://c.y.qq.com/splcloud/fcgi-bin/fcg_get_diss_by_tag.fcg?rnd=0.8914799780923608&g_tk=5381&jsonpCallback=getPlaylist&loginUin=0&hostUin=0&format=jsonp&inCharset=utf8&outCharset=utf-8&notice=0&platform=yqq&needNewCode=0&categoryId=10000000&sortId=5&sin=0&ein=29
```

下面搜集一些处理方式：
### 比如上面的请求链接
```
在请求头里面有1个属性： 
referer:https://y.qq.com/portal/playlist.html

该字段暂时无法处理的。使用开发环境的代理功 + axios 设置请求头来模拟正确的请求头
```

在配置文件dev-server.js中的express
```javascript
var axios = require('axios')
var apiRoutes = express.Router()

apiRoutes.get('/getDiscList', function (req, res) {
  var url = 'https://c.y.qq.com/splcloud/fcgi-bin/fcg_get_diss_by_tag.fcg'
  axios.get(url, {
    headers: {
      referer: 'https://c.y.qq.com/',
      host: 'c.y.qq.com'
    },
    params: req.query    // 代理到的查询参数
  }).then((response) => {
    res.json(response.data)   // 代理获取后返回的res
  }).catch((e) => {
    console.log(e)
  })
})

app.use('/api', apiRoutes)

在调用的时候，就不是jsonp请求了，而是一个普通的ajax请求了。
```



## jsonp
https://github.com/webmodules/jsonp

> 很简单，就是利用`<script>`标签没有跨域限制的“漏洞”（历史遗迹啊）来达到与第三方通讯的目的。
 当需要通讯时，本站脚本创建一个`<script>`元素，地址指向第三方的API网址，形如： `<script src="http://www.example.net/api?param1=1&param2=2"></script>`     并提供一个回调函数来接收数据（函数名可约定，或通过地址参数传递）。     
 第三方产生的响应为json数据的包装（故称之为jsonp，即json padding），形如：     `callback({"name":"hax","gender":"Male"})`     这样浏览器会调用`callback`函数，并传递解析后json对象作为参数。本站脚本可在`callback`函数里处理所传入的数据。    
补充：“历史遗迹”的意思就是，如果在今天重新设计的话，也许就不会允许这样简单的跨域了嘿，比如可能像XHR一样按照CORS规范要求服务器发送特定的http头。

把该库安装到项目中
```bash
npm i jsonp -S

```

因为很多地方都会用到，所以适当的进行工具类的封装
```javascript
import originJSONP from 'jsonp'

export default function jsonp (url, data, option) {
  // 需要判断url后面是否有问号
  url += (url.indexOf('?') < 0 ? '?' : '&') + param(data)
  return new Promise((resolve, reject) => {
    originJSONP(url, option, (err, data) => {
      if (!err) {
        resolve(data)
      } else {
        reject(err)
      }
    })
  })
}

function param (data) {
  let url = ''
  for (var k in data) {
    let value = data[k] !== undefined ? data[k] : ''
    url += `&${k}=${encodeURIComponent((value))}`
  }
  // 如果url有值，则将第一个 & 符号去掉
  return url ? url.substring(1) : ''
}

```

## 流程
1. 先把需要抓去的数据拿到
2. 按照页面一个一个来解决
3. 先解决轮播图，编写轮播图的基础组件 base/slider
4. 然后解决推荐列表
5. 发现整个页面没法滚动，封装bas/scroll组件

[轮播图组件](/chapter/m/recommend/slider.md) 
轮播图搞定之后，发现列表滚动条不了，我们想整轮播图和推荐列表一起被滚动，然后就开始封装滚动组件
[滚动条组件](/chapter/m/recommend/scroll.md)



