# 歌手页
![](/assets/musicapp/歌手页和歌手详情页面.png)
效果图如上：
1. 主屏是歌手列表，右侧边栏是字母排序分类，左右都是联动的，右边滚动左边也会跟着变化，就如同联系人列表一样
2. 歌手详情列表

## 接口的获取

```
QQ音乐pc端歌手页面：
https://y.qq.com/portal/singer_list.html

从请求的页面中找到的列表请求：
https://c.y.qq.com/v8/fcg-bin/v8.fcg?channel=singer&page=list&key=all_all_all&pagesize=100&pagenum=1&g_tk=5381&jsonpCallback=GetSingerListCallback&loginUin=0&hostUin=0&format=jsonp&inCharset=utf8&outCharset=utf-8&notice=0&platform=yqq&needNewCode=0

下面是其中一条数据的结果信息：
{
    "Farea": "1",
    "Fattribute_3": "3",
    "Fattribute_4": "0",
    "Fgenre": "0",
    "Findex": "X",
    "Fother_name": "Joker",
    "Fsinger_id": "5062",
    "Fsinger_mid": "002J4UUk29y8BY",
    "Fsinger_name": "薛之谦",
    "Fsinger_tag": "541,555",
    "Fsort": "1",
    "Ftrend": "0",
    "Ftype": "0",
    "voc": "0"
}

仔细对比qq音乐的页面中的数据和这里的数据，发现没有图片？下面来分析下图片是怎么出来的。

打开页面中已经渲染好的两张图片地址：
https://y.gtimg.cn/music/photo_new/T001R150x150M0000025NhlN2yWrP4.jpg?max_age=2592000
https://y.gtimg.cn/music/photo_new/T001R150x150M000002J4UUk29y8BY.jpg?max_age=2592000

看出来了吗？T001R150x150M000002J4UUk29y8BY.jpg 这一串中的002J4UUk29y8BY和返回的信息里面的Fsinger_mid字段一样，由此可见图片地址是拼接而成的。
```