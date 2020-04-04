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
      <node-content
        :data="node"
        :level="levelData[node[props.key]]"
        :showNext="!foldData[node.id]">
      </node-content>
    </div>
  </div>
</template>

<script>

export default { 
  components: {
    'node-content': {
      props: ['data', 'level', 'showNext'],
      computed: {
        slotData () {
          return {
            data: this.data,
            level: this.level,
            showNext: this.showNext
          }
        }
      },
      render (h) {
        let slot = this.$parent.$scopedSlots.default
        let props = this.$parent.props
        return slot ? slot(this.slotData) : (<div>{ this.data[props.label] }</div>)
      }
    },
  },
  props: {
    treeData: {
      required: true,
    },
    customProps: {
      default: () => ({})
    },
    // 同时渲染的节点数量
    renderNum: {
      default: 100
    },
    // 缓冲区大小，用于节流，避免滚动条上下滚动很短距离就立刻实时计算动态列表
    buffer: {
      default: 25
    }
  },
  data () {
    return {
      foldData: {}, // 被折叠子项 key => true
      levelData: {}, // 节点层级 key => level
      start: 0, // 动态渲染开始下标，preRenderList[start]对应渲染的第一个DOM
      scrollPosition: -1, // 记录滚动条位置
    }
  },
  computed: {
    props () {
      let defaultProps = {
        key: 'id',
        childrenKey: 'children',
        label: 'title'
      }
      return Object.assign({}, defaultProps, this.customProps)
    },
    treeList () {
      return this.treeToList(this.treeData)
    },
    preRenderList () {
      // treeList过滤掉被折叠的子节点
      let list = [], foldLevel = Infinity
      this.treeList.forEach(data => {
        let key = data[this.props.key]
        if (this.levelData[key] <= foldLevel) {
          list.push(data)
          foldLevel = this.foldData[key] ? this.levelData[key] : Infinity
        }
      })
      return list
    },
    renderList () {
      return this.preRenderList.slice(this.start, this.start + this.renderNum)
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
    scroll () {
      this.checkRender()
    },
    checkRender () {
      if (this.scrollPosition >= 0) return
      let len = this.$refs.node.length
      if (!len) return
      this.scrollPosition = this.$el.scrollTop
      this.freezeScrollBar()

      let rect = this.$el.getBoundingClientRect()
      let y = (rect.top + rect.bottom) / 2
      // 在容器最中间的元素
      let centerNode = this.$refs.node.find(n => {
        let r = n.getBoundingClientRect()
        return r.top <= y && r.bottom >= y
      })

      centerNode = centerNode || this.$refs.node[len - 1]
      let identify = centerNode.getAttribute('identify')
      let targetTop = centerNode.getBoundingClientRect().top
      let sourceIdx = this.preRenderList.findIndex(data => String(data[this.props.key]) === identify)
      let preStart = 0
      if (sourceIdx !== -1) {
        preStart = sourceIdx - Math.floor(this.renderNum / 2)
      }
      if (Math.abs(preStart - this.start) < this.buffer) {
        this.unfreezeScrollBar()
        return
      }
      this.start = Math.max(0, preStart)

      // 还原滚动条位置
      this.$nextTick(() => {
        let centerDom = this.$refs.node.find(dom => dom.getAttribute('identify') === identify)
        if (!centerDom) {
          this.unfreezeScrollBar()
          this.checkRender()
          return
        }
        let deltaScroll = centerDom.getBoundingClientRect().top - targetTop
        this.$el.scrollTop += deltaScroll
        this.unfreezeScrollBar()
      })
    },

    freezeScrollBar () {
      window.requestAnimationFrame(() => {
        if (this.scrollPosition >= 0) {
          this.$el.scrollTop = this.scrollPosition
          this.freezeScrollBar()
        }
      })
    },
    unfreezeScrollBar () {
      this.scrollPosition = -1
    },

    /**
     * 对外接口
     */
    showChildren (key, unfold) {
      this.$set(this.foldData, key, !unfold)
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