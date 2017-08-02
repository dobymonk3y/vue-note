# listView歌手列表联/系人列表


## 第一个版本
![](/assets/musicapp/listView歌手列表联-系人列表.png)

上面的列表使用了scoll组件，然后渲染了列表
```html
<template>
  <scroll class="list-view" :data="data">
    <ul>
      <li v-for="group in data" class="list-group">
        <h2 class="list-group-title">{{ group.title }}</h2>
        <ul>
          <li v-for="item in group.items" class="list-group-item">
            <img :src="item.avtar" class="avtar">
            <span class="name">{{item.name}}</span>
          </li>
        </ul>
      </li>
    </ul>
  </scroll>
</template>
```

