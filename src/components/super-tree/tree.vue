<template>
  <div class="super-tree-container" @scroll="scroll">
    <div
      v-for="node of renderList"
      ref="node"
      :key="node.id"
      :identify="node.id"
      :style="{
        paddingLeft: `${levelData[node[props.key]] * 20}px`
      }"
      class="node">
      <span v-if="foldData[node.id]">展开</span>{{ node.title }}
    </div>
  </div>
</template>

<script>
function generateTree () {
  console.time('ge')
  let map = {}
  let result = []
  for (let i = 0; i < 100000; i++) {
    let list = new Array(Math.floor(Math.random() * 40)).fill('hello')
    let data = {
      title: `节点${i}${list.join('')}`,
      id: i,
      parentId: undefined,
      children: []
    }
    map[i] = data
    if (result.length > 300000) {
      let length = Object.keys(map).length - 1
      data.parentId = Math.floor(Math.random() * length)
      let parent = map[data.parentId]
      parent.children.push(data)
    } else {
      result.push(data)
    }
  }
  console.timeEnd('ge')
  return result
}

export default {
  props: {
    treeData: {
      default: () => generateTree()
    },
    props: {
      default: () => ({
        key: 'id',
        childrenKey: 'children'
      })
    },
    renderNum: {
      default: 100
    },
    buffer: {
      default: 25
    }
  },
  data () {
    return {
      foldData: {},
      levelData: {},
      start: 0,
      preStart: 0,
    }
  },
  computed: {
    treeList () {
      console.time('treeToList')
      let res = this.treeToList(this.treeData)
      console.timeEnd('treeToList')
      return res
    },
    renderList () {
      return this.treeList.slice(this.start, this.start + this.renderNum)
    }
  },
  created () {
  },
  methods: {
    treeToList (tree, result = [], level = 0) {
      tree.forEach(node => {
        result.push(node)
        this.$set(this.levelData, node[this.props.key], level + 1)
        node[this.props.childrenKey] && this.treeToList(node[this.props.childrenKey], result, level + 1)
      })
      return result
    },

    checkRender () {
      this.$nextTick(() => {
        if (!this.$refs.node.length) return
        let rect = this.$el.getBoundingClientRect()
        let y = (rect.top + rect.bottom) / 2
        // 在容器最中间的元素
        let scrollTop = this.$el.scrollTop
        let centerNode = this.$refs.node.find(n => {
          let r = n.getBoundingClientRect()
          return r.top <= y && r.bottom >= y
        })
        if (!centerNode) {
          centerNode = this.$refs.node[this.$refs.node.length - 1]
        }
        let identify = centerNode.getAttribute('identify')
        let targetTop = centerNode.getBoundingClientRect().top
        let sourceIdx = this.treeList.findIndex(data => String(data[this.props.key]) === identify)
        let oldStart = this.start
        let preStart = 0
        if (sourceIdx !== -1) {
          preStart = sourceIdx - Math.floor(this.renderNum / 2)
        }
        if (Math.abs(preStart - this.start) >= this.buffer) {
          this.start = Math.max(0, preStart)
        }
        if (oldStart === this.start) return
        // 还原滚动条位置
        this.$nextTick(() => {
          let centerDom = this.$refs.node.find(dom => dom.getAttribute('identify') === identify)
          if (!centerDom) {
            window.requestAnimationFrame(this.checkRender)
            return
          }
          let deltaScroll = centerDom.getBoundingClientRect().top - targetTop
          this.$el.scrollTop = scrollTop + deltaScroll
        })
      })
    },
    scroll () {
      window.requestAnimationFrame(this.checkRender)
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
  .node
    padding 5px 10px
</style>