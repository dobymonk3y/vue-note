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

请求数据得到了，但是，这个返回的数据并不符合我们页面设计的需求，需要加工一下:
以下代码学习到的新知识是：

1. 使用数组的sort排序
2. 使用数组的concat把两个数组合并
```javascript
      /**
       * 规范化歌手数据；
       * 原因：
       * api直接获取到的不符合我们的页面逻辑的数据
       * 我们的列表是一个按字母列表分组的数据
       * 所以要按字母分组聚合
       * @param list
       * @private
       */
      _normalizeSinger (list) {
        let map = {
          // 热门数据取前10
          hot: {
            title: HOT_NAME,
            items: []
          }
        }
        list.forEach((item, index) => {
          if (index < HOT_SINGER_LEN) {
            map.hot.items.push(new Singer({
              id: item.Fsinger_id,
              name: item.Fsinger_name,
              mid: item.Fsinger_mid
            }))
          }
          // 按 index的字母聚合数据
          const key = item.Findex
          if (!map[key]) {
            map[key] = {
              title: key,
              items: []
            }
          }
          map[key].items.push(new Singer({
            id: item.Fsinger_id,
            name: item.Fsinger_name,
            mid: item.Fsinger_mid
          }))
        })
        // 为了得到有序列表，我们需要处理 map
        let hot = []
        let ret = []
        for (let key in map) {
          let val = map[key]
          // 如果是字母的话就提取出来，因为还看到map中还有数字的index，这个数据是不需要的
          if (val.title.match(/[a-zA-Z]/)) {
            ret.push(val)
          } else if (val.title === HOT_NAME) {
            hot.push(val)
          }
        }
        // 升序排序
        ret.sort((a, b) => {
          // 得到字母的charcode 然后对比
          return a.title.charCodeAt(0) - b.title.charCodeAt(0)
        })
        // 再把两个数组链接起来,合并成一个
        return hot.concat(ret)
      }
    }
```