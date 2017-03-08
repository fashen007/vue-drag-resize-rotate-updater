a vue2 component. Taking a cue from react-resizable and react-draggable.

![DEMO](https://raw.githubusercontent.com/tigerlove/vue-drag-resize-rotate/master/screenshot.gif)

## Install
```
npm install vue-drag-resize-rotate-updater

```
##声明：这个非原创,因为[原作者](https://github.com/tigerlove/vue-drag-resize-rotate)没有及时修复bug我又急用所以借用了
## Usage
```
<template>
<div>
  <vue-drr :w='180' :h='180' :r='0' :rotatable='false' v-on:change="showchange">
    I can be dragged anywhere
  </vue-drr>
  <div style="height: 500px; width: 500px;float:left;position: relative;border: 1px solid black;padding:5px">
    <vue-drr :w='180' :h='180' :bounds='{parent:true}' :rotatable='true'>
      <div>I can only be moved in any parent.</div>
    </vue-drr>
  </div>
  <!-- 	<vue-drr :w='180' :h='180' :r='30' :rotatable='true'>
  		 rotated 30deg at the begaining
      </vue-drr> -->
  <vue-drr :w='180' :h='180' axis='x' :rotatable='true'>
    I can only be dragged horizonally
  </vue-drr>
  <vue-drr :w='180' :h='180' axis='y' :rotatable='true'>
    I can only be dragged vertically
  </vue-drr>
  <vue-drr :w='180' :h='180' handle='.dragme' :rotatable='true'>
    <div class='dragme' style='background: #ccc;border: 1px solid black;padding:5px'>drag here</div>
    <div>lalalalaala</div>
  </vue-drr>
  <vue-drr :w='180' :h='180' cancel='.dnotdragme' :rotatable='true'>
    <div class='dnotdragme' style='background: #ccc;border: 1px solid black;padding:5px'>don't drag here</div>
    <div>lalalalaala</div>
  </vue-drr>
  <vue-drr :w='180' :h='180' :grid='[50,50]' :rotatable='true'>

    <div>snap to 50 * 50</div>
  </vue-drr>
  <vue-drr :w='180' :h='180' :bounds='{top: -100, left: -100, right: 100, bottom: 100}' :rotatable='true'>
    <div>I can only be moved 100px in any direction.</div>
  </vue-drr>
</div>
</template>
<style>

</style>
<script>
import VueDRR from 'vue-drag-resize-rotate'
export default {
  name: 'app',
  components: {
    'vue-drr': VueDRR
  },
  methods: {
    showchange: function (data) {
      console.log(JSON.stringify(data))
    }
  }
}
</script>
```
###APi
```
w: 表示默宽度                        Number
h: 表示默认高度                      Number
x: 表示默认距离x轴原点的距离           Number
y: 表示默认距离y轴原点的距离           Number
axis: 表示能锁定移动轴                 String
rotatable: 表示是否能够旋转，默认可以    Boolean

//
handle: 表示可以拖拽的子集class         String
cancel： 表示不能拖拽的子集class        String

grid : 表示移动的跨度                  Array  // [10, 10] 表示如果在x轴上移动一次就是10像素 y轴同理

bounds： 表示能移动的范围               Object // :bounds='{top: -100, left: -100, right: 100, bottom: 100}'
```
