<style scoped>
.dragable {
  position: absolute;
  text-align: center;
  background: #fff;
  box-sizing: border-box;
}
.is-active {
  border: 1px solid #20a0ff;
}
.scale {
  position: absolute;
  width: 10px;
  height: 10px;
  bottom: 0;
  right: 0;
  border: 1px solid #999;
  background-color: #eee;
  background-position: bottom right;
  padding: 0 3px 3px 0;
  background-repeat: no-repeat;
  background-origin: content-box;
  box-sizing: border-box;
  cursor: se-resize;
}

.rotate {
  position: absolute;
  width: 20px;
  height: 20px;
  left: 50%;
  margin-left: -10px;
  bottom: -20px;
  background-color: #000;
  background-position: top right;
  padding: 3px 3px 0px 0;
  background-repeat: no-repeat;
  background-origin: content-box;
  box-sizing: border-box;
  cursor: alias;
}
.scale-top-left {
  cursor: nwse-resize;
  top: 5px;
  left: 5px;
}
.scale-top-right {
  cursor: nesw-resize;
  top: 5px;
  right: 5px;
}
.scale-bottom-left {
  bottom: 5px;
  left: 5px;
  cursor: nesw-resize;
}
.scale-bottom-right {
  bottom: 5px;
  right: 5px;
  cursor: nwse-resize;
}
</style>
<template>
  <div class='dragable' :class="{'is-active': isActive}" @mousedown='handleDown' @mouseup.stop='handleUp' :style='boxStyle'>
    <slot></slot>
    <template v-if='this.isActive'>
      <div class='scale scale-top-left' @mousedown.stop='(e) => { resizeStart(e, "nw")}'></div>
      <div class='scale scale-top-right' @mousedown.stop='(e) => { resizeStart(e, "ne")}'></div>
      <div class='scale scale-bottom-left' @mousedown.stop='(e) => { resizeStart(e, "sw")}'></div>
      <div class='scale scale-bottom-right' @mousedown.stop='(e) => { resizeStart(e, "se")}'></div>
      <div class='rotate' @mousedown.stop='rotateStart' v-show='rotatable'></div>
    </template>
  </div>
</template>

<script>
import Vector from 'victor'

let findInArray = function(array, callback) {
  for (let i = 0, length = array.length; i < length; i++) {
    if (callback.apply(callback, [array[i], i, array])) return array[i]
  }
}

let isFunction = function(func) {
  return typeof func === 'function' || Object.prototype.toString.call(func) === '[object Function]'
}

let isNum = function(num) {
  return typeof num === 'number' && !isNaN(num)
}

let int = function(a) {
  return parseInt(a, 10)
}

let outerHeight = function(node) {
  // This is deliberately excluding margin for our calculations, since we are using
  // offsetTop which is including margin. See getBoundPosition
  let height = node.clientHeight
  let computedStyle = window.getComputedStyle(node)
  height += int(computedStyle.borderTopWidth)
  height += int(computedStyle.borderBottomWidth)
  return height
}

let outerWidth = function(node) {
  // This is deliberately excluding margin for our calculations, since we are using
  // offsetLeft which is including margin. See getBoundPosition
  let width = node.clientWidth
  let computedStyle = window.getComputedStyle(node)
  width += int(computedStyle.borderLeftWidth)
  width += int(computedStyle.borderRightWidth)
  return width
}
let innerHeight = function(node) {
  let height = node.clientHeight
  let computedStyle = window.getComputedStyle(node)
  height -= int(computedStyle.paddingTop)
  height -= int(computedStyle.paddingBottom)
  return height
}

let innerWidth = function(node) {
  let width = node.clientWidth
  let computedStyle = window.getComputedStyle(node)
  width -= int(computedStyle.paddingLeft)
  width -= int(computedStyle.paddingRight)
  return width
}

let matchesSelectorFunc = ''
let matchesSelector = function(el, selector) {
  if (!matchesSelectorFunc) {
    matchesSelectorFunc = findInArray([
      'matches',
      'webkitMatchesSelector',
      'mozMatchesSelector',
      'msMatchesSelector',
      'oMatchesSelector'
    ], function(method) {
      return isFunction(el[method])
    })
  }

  return el[matchesSelectorFunc].call(el, selector)
}

let getBoundPosition = function(bounds, node, clientX, clientY) {
  // If no bounds, short-circuit and move on
  if (!bounds) return [clientX, clientY]

  let parent = node.parentNode

  if (bounds.parent) {
    let nodeStyle = window.getComputedStyle(node)
    let parentStyle = window.getComputedStyle(parent)
      // Compute bounds. This is a pain with padding and offsets but this gets it exactly right.
    bounds = {
      left: -node.offsetLeft + int(parentStyle.paddingLeft) +
        int(nodeStyle.borderLeftWidth) + int(nodeStyle.marginLeft),
      top: -node.offsetTop + int(parentStyle.paddingTop) +
        int(nodeStyle.borderTopWidth) + int(nodeStyle.marginTop),
      right: innerWidth(parent) - node.offsetLeft - outerWidth(node),
      bottom: innerHeight(parent) - outerHeight(node) - node.offsetTop
    }
  }

  // Keep x and y below right and bottom limits...
  if (isNum(bounds.right)) clientX = Math.min(clientX, bounds.right)
  if (isNum(bounds.bottom)) clientY = Math.min(clientY, bounds.bottom)

  // But above left and top limits.
  if (isNum(bounds.left)) clientX = Math.max(clientX, bounds.left)
  if (isNum(bounds.top)) clientY = Math.max(clientY, bounds.top)

  return [clientX, clientY]
}

export default {
  replace: true,
  name: 'dragable',
  props: {
    w: {
      type: Number,
      default: 40
    },
    h: {
      type: Number,
      default: 40
    },
    x: {
      type: Number,
      default: 0
    },
    y: {
      type: Number,
      default: 0
    },
    r: {
      type: Number,
      default: 0
    },
    rotatable: {
      type: Boolean,
      default: true
    },
    axis: {
      type: String,
      default: 'both'
    },
    handle: {
      type: String,
      default: ''
    },
    cancel: {
      type: String,
      default: ''
    },
    grid: {
      type: Array,
      default: function() {
        return [0, 0]
      }
    },
    bounds: {
      type: Object,
      default: null
    },
    isActive: {
      type: Boolean,
      default: false
    },
    uid: {
      default: ''
    },
    sameSizeRatio: true // 是否等比例缩放
  },
  mounted: function() {
    var el = document.documentElement
    var event = 'mousemove'
    var handler = this.handleMove

    if (el.attachEvent) {
      el.attachEvent('on' + event, handler)
    } else if (el.addEventListener) {
      el.addEventListener(event, handler, true)
    } else {
      el['on' + event] = handler
    }

    if (!this.rotatable) {
      this.$el.getElementsByClassName('rotate')[0].className = ''
    } else {
      this.rotate = this.r
    }
    // doucment下mouseup都会释放拖拽或者缩放事件
    document.addEventListener('mouseup', this.handleUp)
  },
  beforeDestroy: function() {
    var el = document.documentElement
    var event = 'mousemove'
    var handler = this.handleMove

    if (el.detachEvent) {
      el.detachEvent('on' + event, handler)
    } else if (el.removeEventListener) {
      el.removeEventListener(event, handler, true)
    } else {
      el['on' + event] = null
    }
  },
  methods: {
    getRotateAngle: function(x, y) {
      var rCenter = this._getCenter()
      var vStart = new Vector(this.rotateStartX - rCenter.x, this.rotateStartY - rCenter.y)
      var vEnd = new Vector(x - rCenter.x, y - rCenter.y)
      return vEnd.angleDeg() - vStart.angleDeg()
    },
    _getCenter: function() {
      var rect = this.$el.getBoundingClientRect()
      return {
        x: (rect.left + rect.right) / 2,
        y: (rect.bottom + rect.top) / 2
      }
    },
    rotateStart: function(e) {
      if (this.rotatable) {
        this.rotateStartX = e.clientX
        this.rotateStartY = e.clientY
        this.rotating = true
      }
    },
    resizeStart: function(e, direction) {
      this.direction = direction // 方向
      this.resizeStartX = e.clientX
      this.resizeStartY = e.clientY
      this.resizing = true
      this.dragging = false
      this.lastW = this.localw
      this.lastH = this.localh
    },
    handleDown: function(e) {
      if (this.handle && !matchesSelector(e.target, this.handle)) {
        return
      }
      if (this.cancel && matchesSelector(e.target, this.cancel)) {
        return
      }
      if (!this.lastX) {
        this.lastX = e.clientX
        this.lastY = e.clientY
      }
      this.dragging = true
    },
    handleUp: function(e) {
      // 如果没有做任何更改也没必要向父级emit事件
      console.log(1)
      if (!this.dragging && !this.resizing && !this.rotating) {
        this.$emit('handleUp', {
          uid: ''
        })
        return
      }
      this.dragging = false
      this.resizing = false
      this.rotating = false
      this.$emit('handleUp', {
        x: this.localx,
        y: this.localy,
        w: this.localw,
        h: this.localh,
        r: this.localr,
        uid: this.uid
      })
    },
    handleMove: function(e) {
      if (e.stopPropagation) e.stopPropagation()
      if (e.preventDefault) e.preventDefault()
      if (this.dragging && this.isActive) {
        let deltax = e.clientX - this.lastX
        let deltay = e.clientY - this.lastY

        let deltaxround = Math.round(deltax / this.grid[0]) * this.grid[0]
        let deltayround = Math.round(deltay / this.grid[1]) * this.grid[1]

        let thisx = this.localx
        let thisy = this.localy

        if (this.grid[0] > 0 && this.grid[1] > 0) {
          if (this.axis === 'both') {
            thisx = deltaxround
            thisy = deltayround
          } else if (this.axis === 'x') {
            thisx = deltaxround
          } else if (this.axis === 'y') {
            thisy = deltayround
          }
        } else {
          if (this.axis === 'both') {
            thisx = e.clientX - this.lastX
            thisy = e.clientY - this.lastY
          } else if (this.axis === 'x') {
            thisx = e.clientX - this.lastX
          } else if (this.axis === 'y') {
            thisy = e.clientY - this.lastY
          }
        }

        if (this.bounds) {
          [thisx, thisy] = getBoundPosition(this.bounds, this.$el, thisx, thisy)
        }
        this.localx = thisx
        this.localy = thisy
      }
      if (this.resizing) {
        let distanceX = 0
        let distanceY = 0
        if (this.direction === 'nw' || this.direction === 'sw') {
          distanceX = parseInt(this.resizeStartX) - parseInt(e.clientX)
          distanceY = parseInt(this.resizeStartY) - parseInt(e.clientY)
        } else {
          distanceX = parseInt(e.clientX) - parseInt(this.resizeStartX)
          distanceY = parseInt(e.clientY) - parseInt(this.resizeStartY)
        }
        this.localw = parseInt(this.lastW) + distanceX
        this.localh = this.sameSizeRatio ? this.lastH / this.lastW * this.localw : parseInt(this.lastH) + distanceY
        if (this.direction === 'nw') { // resize 左上角
          this.localx = e.clientX
          // 当时左上角的时候，这个时候不能直接设置 localy = e.clientY 因为e.clienY并不一定等于高度增加量
          // localy 表示新的坐标，而 新坐标 = 旧坐标 + Y轴增加量
          // 其实就是 一切变化由x轴来决定 在 等比例变化的情况下
          this.localy = this.resizeStartY - (this.localh - this.lastH)
        } else if (this.direction === 'ne') { // resize 右上角
          this.localy = this.resizeStartY - (this.localh - this.lastH) // 同上解释
        } else if (this.direction === 'sw') { // resize 左下角
          this.localx = e.clientX
        }
      }
      if (this.rotating) {
        this.localr = parseInt(this.r) + this.getRotateAngle(e.clientX, e.clientY)
      }
    }
  },

  data: function() {
    return {
      localx: this.x,
      localy: this.y,
      localw: this.w,
      localh: this.h,
      localr: this.r,
      lastX: 0,
      lastY: 0,
      dragging: false,
      rotate: 0,
      resizeStartX: 0,
      resizeStartY: 0,
      rotateStartX: 0,
      rotateStartY: 0,
      resizing: false
    }
  },
  computed: {
    boxStyle: function() {
      let styleObject = {
        width: `${this.localw}px`,
        height: `${this.localh}px`,
        transform: `translate(${this.localx}px, ${this.localy}px) rotate(${this.localr}deg)`
      }
      return styleObject
    }
  }
}
</script>
