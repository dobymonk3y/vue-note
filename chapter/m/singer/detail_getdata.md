# 歌手详情页-歌手列表数据抓取

上一节中，通过路由跳转和vuex把参数都传递过来了，那么就先把列表数据抓取到。

同样的先去qq音乐官网：https://y.qq.com/n/yqq/singer/002J4UUk29y8BY.html# 查看该页的请求，找到 歌曲列表的接口，然后封装得到这个接口数据

他们家的接口数据一般都是.fcg结尾的所以不是很难找。接口如下`https://c.y.qq.com/v8/fcg-bin/fcg_v8_singer_track_cp.fcg?g_tk=5381&jsonpCallback=MusicJsonCallbacksinger_track&loginUin=0&hostUin=0&format=jsonp&inCharset=utf8&outCharset=utf-8&notice=0&platform=yqq&needNewCode=0&singermid=002J4UUk29y8BY&order=listen&begin=0&num=30&songstatus=1`

数据列表通过jsonp请求到之后，我们需要对数据进行处理。（鉴于后面的多个模块都需要歌曲类的信息也应该进行抽象了）
## Song类的封装

**歌曲真实播放地址获取：**
在返回的数据里面全部没有地址，地址都是固定的接口，然后使用各种参数拼接而成的。获取方法如下：

点击一首歌曲播放，会跳转到一个播放器的页面，这个时候是看不到请求地址的，我们要把谷歌浏览器的控制台打开，然后不关闭当前的页面，在之前的页面中再重复一次操作，点击播放，就能在播放器页面中的请求地址中和控制台中有打印如下：
请求地址中的：
```
http://dl.stream.qqmusic.qq.com/C400003v4UL61IYlTY.m4a?vkey=E30D4E86BAA34DE111F6B8E537FE7B9AF1A03CE80B6A9F9B4361E6295B38EBC7FD22D70A28D7CBF608B93F8FF590AA8E4D977410B1F63874&guid=6078336128&uin=0&fromtag=66
```
控制台打印的：
```
Mixed Content: The page at 'https://y.qq.com/portal/player.html' was loaded over HTTPS, but requested an insecure video 'http://dl.stream.qqmusic.qq.com/C400003v4UL61IYlTY.m4a?vkey=E30D4E86BAA34DE111F6B8E537FE7B9AF1A03CE80B6A9F9B4361E6295B38EBC7FD22D70A28D7CBF608B93F8FF590AA8E4D977410B1F63874&guid=6078336128&uin=0&fromtag=66'. This content should also be served over HTTPS.
```

然后就能看到里面有一个m4a的请求，然而我尝试拼接参数的时候，发现后面的查询参数不知道怎么出来的。最后还是视频中的地址说用songid。

然后测试出来播放地址如下:
```
`http://dl.stream.qqmusic.qq.com/${musicData.songId}.m4a?fromtag=66`
```
