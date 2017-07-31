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
```