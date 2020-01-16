<script>
import Vue from 'vue'
import Popper from './Popper'
const escape = function(text) {
  return text.replace(/<|>|`|"|&/g, '?').replace(/\r\n|\r|\n/g, '<br>')
}

export default {
  name: 'vue-mention',
  props: {},
  components: {},
  $portal: null,
  vm: null,
  data() {
    return {
      triggerDisable: false,
      status: {
        show: false,
        x: 0,
        y: 0,
      },
    }
  },
  created() {
    // console.log(this)
  },
  computed: {
    watchSign() {
      return `${this.status.x}-${this.status.y}`
    },
  },
  render() {
    return (
      <textarea
        onKeyup={this.onChange}
        ref="container"
        class="mentions"
        onCompositionstart={this.onComStart}
        onCompositionend={this.onComEnd}
        rows={10}
        onBlur={this.closePortal}
      >
      </textarea>
    )
  },
  methods: {
    getContainerEl() {
      return this.$refs.container
    },
    onChange(ev) {
      if (this.triggerDisable) return
      const $ = this.getContainerEl()
      const cursorPos = $.selectionStart
      const valueBeforeCursor = ev.target.value.slice(0, cursorPos)
      const matchRes = /#([^\s\b]+)$/g.exec(valueBeforeCursor)

      if (matchRes) {
        const cursorIndex = $.selectionStart
        const beforeText = $.value.slice(0, cursorIndex)
        const afterText = $.value.slice(cursorIndex)
        const style = window.getComputedStyle($)
        const rect = $.getBoundingClientRect()
        const $mirrorEl = document.createElement('div')

        $mirrorEl.innerHTML = `${escape(beforeText)}<span id="__cursor__">|</span>${escape(afterText)}`
        $mirrorEl.style = `${style.cssText}position: fixed;top:${rect.top}px;left:${rect.left}px;`

        document.body.appendChild($mirrorEl)
        const $cursor = document.getElementById('__cursor__')
        const { left, top } = $cursor.getBoundingClientRect()

        this.status.show = true
        this.status.x = left
        this.status.y = top
        $mirrorEl.parentNode.removeChild($mirrorEl)
      }
    },
    onComStart() {
      this.triggerDisable = true
    },
    onComEnd() {
      this.triggerDisable = false
    },
    closePortal() {
      this.vm.$el.parentNode.removeChild(this.vm.$el)
      this.vm = null
      this.$portal = null
    },
    createPortal() {
      if (this.vm) {
        this.vm.x = this.status.x
        this.vm.y = this.status.y
        return
      }
      const { x, y } = this.status
      this.$portal = document.createElement('div')
      this.$portal.id = '__portal__'
      document.body.appendChild(this.$portal)

      this.vm = new Vue({
        data() { return { x, y } },
        render: function() { return <Popper x={this.x} y={this.y}/> },
      })
      this.vm.$mount('#__portal__')
    },
  },
  watch: {
    watchSign(show) {
      if (show) {
        this.createPortal()
      } else {
        this.closePortal()
      }
    },
  },
}

</script>

<style lang="scss">
  textarea {
    width: 600px;
    color: red;
    font-size: 14px;
    line-height: 1.6;
  }

  .__portal__ {
    background: #FFF;
    border: 1px solid #ccc;
  }
</style>
