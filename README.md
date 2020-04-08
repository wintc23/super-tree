# super-tree

## 组件介绍
本组件为解决大量树形/列表数据的渲染而生，它最核心的功能是提供了数据动态列表渲染的方案，组件通过插槽的方式给使用者保留了极高的自由度————你可以自由控制每个节点渲染的文本/菜单，包括展开或折叠的按钮。

## 在你的项目中使用本组件

### 安装组件
```
yarn add vue-super-tree
```
或者
```bash
npm install vue-super-tree
```

### 引入组件
```bash
import { superTree } from 'vue-super-tree'
```
引入后在你需要使用的地方注册为组件即可使用。

### 组件属性
- treeData 数组，树形数据，必传
- customProps 对象，可选，用于告知组件应该
- renderNum 数字，渲染出来的节点数量，渲染的节点至少需要超出一屏或者树节点容器的高度，默认100
- buffer 数字，缓冲区，应该小于(渲染的节点 - 容器内可见的节点数量)/ 2，默认为25。滚动时并非发生滚动就立即更新渲染的节点列表，

### 组件方法
- showChildren(id, isShow) 展开/隐藏某个id的子级
- getNodeMap 获取组件映射表，id => data的映射，可以快速获取某个id对应的节点数据，比如获取某个节点data的父节点：
  该函数返回的对象不应该被一直保存，应该在需要时再去获取，因为如果treeData发生改变，该函数返回的对象也会发生变化。
  ```js
  let parent
  if (data.parentId !== undefined) {
    let nodeMap = this.$refs.tree.getNodeMap()
    parent = nodeMap[data.parentId]
  }

  ```

### 组件插槽
- 组件需要传入一个插槽，以自定义节点渲染。插槽作用域内可以使用 { data, showNext }等参数，分别是节点对应的数据、当前节点的下级是否显示等

### 使用需要传递插槽, 使用示例
```html
<template>
  <tree :treeData="treeData" ref="tree" class="super-tree-container">
    <template v-slot="{ data, showNext }">
      <div class="node-render">
        <div class="icon-container">
          <span
            :class="{
              icon: true,
              'show-next': !showNext,
              'no-next': !data.children || !data.children.length
            }"
            @click.stop="showChildren(data.id, !showNext)">
          </span>
        </div>
        <div class="title">{{ data.title }}</div>
      </div>
    </template>
  </tree>

</template>

<script>
import { superTree } from 'vue-super-tree'

// 随机生成树结构
function generateTree () {
  console.time('ge')
  let map = new Map()
  let result = []
  for (let i = 0; i < 10000; i++) {
    let list = new Array(Math.floor(Math.random() * 100)).fill('a')
    let data = {
      title: `节点${i}--${list.join('')}`,
      id: i,
      parentId: undefined,
      children: []
    }
    map.set(i, data)
    if (result.length > 3000) {
      let length = map.size - 1
      data.parentId = Math.floor(Math.random() * length)
      let parent = map.get(data.parentId)
      parent.children.push(data)
    } else {
      result.push(data)
    }
  }
  console.timeEnd('ge')
  return result
}

export default {
  components: {
    Tree: superTree
  },
  data () {
    return {
      treeData: []
    }
  },
  created () {
    this.treeData = generateTree()
  },
  methods: {
    showChildren (id, show) {
      this.$refs.tree.showChildren(id, show)
    }
  }
}
</script>

<style lang="stylus" scoped>
.super-tree-container
  word-break break-all
  position relative
  text-align left
  height 80vh
  border-radius 8px
  background #eee
  overflow auto

.node-render
  padding 5px 10px
  display flex
  .icon-container
    flex-shrink 0
    display flex
    align-items center
    padding 0 10px
    .icon
      display flex
      align-items center
      justify-content center
      cursor pointer
      height 1em
      width 1em
      &::before
        content ''
        border 6px solid transparent
        border-top-color #ffa741
        border-bottom-width 0
        transition transform .1s ease-out
      &.show-next::before
        transform rotate(-90deg)

      &.no-next::before
        border 4px solid #ffa741
        border-radius 50%

</style>

```





## 源码运行


```bash
yarn # 安装依赖
yarn serve # 启动开发服务
```

