<template>
  <div id="app">
    <tree :treeData="treeData" ref="tree">
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
  </div>
</template>

<script>
import { superTree } from 'vue-super-tree'

function generateTree () {
  console.time('ge')
  let map = new Map()
  let result = []
  for (let i = 0; i < 30000; i++) {
    let list = new Array(Math.floor(Math.random() * 100)).fill('a')
    let data = {
      title: `节点${i}--${list.join('')}`,
      id: i,
      parentId: undefined,
      children: []
    }
    map.set(i, data)
    if (result.length > 1000) {
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
  name: 'App',
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

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
html {
  /* background: #333; */
}
</style>

<style lang="stylus" scoped>
.node-render
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
