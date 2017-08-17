# 歌手详情页-歌手列表数据抓取

上一节中，通过路由跳转和vuex把参数都传递过来了，那么就先把列表数据抓取到。

同样的先去qq音乐官网：https://y.qq.com/n/yqq/singer/002J4UUk29y8BY.html# 查看该页的请求，找到 歌曲列表的接口，然后封装得到这个接口数据

他们家的接口数据一般都是.fcg结尾的所以不是很难找。接口如下`https://c.y.qq.com/v8/fcg-bin/fcg_v8_singer_track_cp.fcg?g_tk=5381&jsonpCallback=MusicJsonCallbacksinger_track&loginUin=0&hostUin=0&format=jsonp&inCharset=utf8&outCharset=utf-8&notice=0&platform=yqq&needNewCode=0&singermid=002J4UUk29y8BY&order=listen&begin=0&num=30&songstatus=1`

数据列表通过jsonp请求到之后，我们需要对数据进行处理。
## Song类的封装

