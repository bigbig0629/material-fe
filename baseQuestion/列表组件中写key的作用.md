基于没有 key 的情况，diff 速度会更快
没有绑定 key 的情况下，并且在遍历模板简单的情况下，会导致虚拟新旧节点对比更快，节点也会复用（就地复用）

#### 举例

```html
<div id="app">
  <div v-for="i in dataList">{{ i }}</div>
</div>
```

```javaScript
var vm = new Vue({
  el: '#app',
  data: {
    dataList: [1, 2, 3, 4, 5]
  }
})
```

以上的例子，v-for 的内容会生成以下的 dom 节点数组，我们给每一个节点标记一个身份 id：

```javascript
[
  "<div>1</div>", // id： A
  "<div>2</div>", // id:  B
  "<div>3</div>", // id:  C
  "<div>4</div>", // id:  D
  "<div>5</div>", // id:  E
];
```

1. 改变 dataList 数据，进行数据位置替换，对比改变后的数据

```javascript
vm.dataList = [4, 1, 3, 5, 2][ // 数据位置替换
  // 没有key的情况， 节点位置不变，但是节点innerText内容更新了
  ("<div>4</div>", // id： A
  "<div>1</div>", // id:  B
  "<div>3</div>", // id:  C
  "<div>5</div>", // id:  D
  "<div>2</div>") // id:  E
][
  // 有key的情况，dom节点位置进行了交换，但是内容没有更新
  // <div v-for="i in dataList" :key='i'>{{ i }}</div>
  ("<div>4</div>", // id： D
  "<div>1</div>", // id:  A
  "<div>3</div>", // id:  C
  "<div>5</div>", // id:  E
  "<div>2</div>") // id:  B
];
```

2. 增删 dataList 列表项

```javascript
vm.dataList = [3, 4, 5, 6, 7][ // 数据进行增删
  // 1. 没有key的情况， 节点位置不变，内容也更新了
  ("<div>3</div>", // id： A
  "<div>4</div>", // id:  B
  "<div>5</div>", // id:  C
  "<div>6</div>", // id:  D
  "<div>7</div>") // id:  E
][
  // 2. 有key的情况， 节点删除了 A, B 节点，新增了 F, G 节点
  // <div v-for="i in dataList" :key='i'>{{ i }}</div>
  ("<div>3</div>", // id： C
  "<div>4</div>", // id:  D
  "<div>5</div>", // id:  E
  "<div>6</div>", // id:  F
  "<div>7</div>") // id:  G
];
```

---

- 从以上来看，不带 key，并且使用简单的额模板，就这个前提下，可以更有效的复用节点，diff 速度来看也是不带 key 更加快速的，因为带 key 在增删节点上有耗时。这就是 vue 文档所说的默认模式。
- **_但是这个并不是 key 作用_**，而是没有 key 的情况下可以对节点就地复用，提高性能。
- 这个默认的模式是高效的，但是只适用于不依赖子组件状态或临时 DOM 状态 (例如：表单输入值) 的列表渲染输出

#### 那么 key 的作用是什么？

> key 是给每个 vnode 节点的唯一 id，可以依靠 key，更准确、更快的拿到 oldVnode 中对应的 vnode 节点

1. 更准确
   因为带 key 就不是就地复用了，在 sameNode 函数 a.key === b.key 对比中可以避免就地复用的情况。所以会更加准确。
2. 更快
   利用 key 的唯一性生成 map 对象来获取对应节点，比遍历方式更快。
