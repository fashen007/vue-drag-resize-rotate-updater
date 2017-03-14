<template>
<div class='main'>
  <vue-drr :w='180' :h='180' :r='0' :rotatable='true' v-on:handleUp="showchange" :isActive='currentUid == 1' uid='1' :sameSizeRatio='true'>
    I can be dragged anywhere
  </vue-drr>
  <!-- <div style="height: 500px; width: 500px;float:left;position: relative;border: 1px solid black;padding:5px" >
    <vue-drr :w='180' :h='180' :bounds='{parent:true}' :rotatable='true' uid='2' :isActive='currentUid == 2'>
      <div>I can only be moved in any parent.</div>
    </vue-drr>
  </div>
	<vue-drr :w='180' :h='180' :r='30' :rotatable='true'>
		 rotated 30deg at the begaining
    </vue-drr>
  <vue-drr :w='180' :h='180' axis='x' :rotatable='true' uid='3' :isActive='currentUid == 3'>
    I can only be dragged horizonally
  </vue-drr>
  <vue-drr :w='180' :h='180' axis='y' :rotatable='true' uid='4' :isActive='currentUid == 4'>
    I can only be dragged vertically
  </vue-drr>
  <vue-drr :w='180' :h='180' handle='.dragme' :rotatable='true' uid='5' :isActive='currentUid == 5'>
    <div class='dragme' style='background: #ccc;border: 1px solid black;padding:5px'>drag here</div>
    <div>lalalalaala</div>
  </vue-drr>
  <vue-drr :w='180' :h='180' cancel='.dnotdragme' :rotatable='true' uid='6' :isActive='currentUid == 6'>
    <div class='dnotdragme' style='background: #ccc;border: 1px solid black;padding:5px'>don't drag here</div>
    <div>lalalalaala</div>
  </vue-drr>
  <vue-drr :w='180' :h='180' :grid='[50,50]' :rotatable='true' uid='7' :isActive='currentUid == 7'>
    <div>snap to 50 * 50</div>
  </vue-drr>
  <vue-drr :w='180' :h='180' :bounds='{top: -100, left: -100, right: 100, bottom: 100}' :rotatable='true' uid='8' :isActive='currentUid == 8'>
    <div>I can only be moved 100px in any direction.</div>
  </vue-drr> -->
</div>
</template>
<style>
html,body {
  padding: 0;
  margin: 0;
  height: 100%;
  width: 100%;
  overflow: hidden;
}
.main {
  height: 100%;
  width: 100%;
  position: relative;
  overflow: hidden;
}
</style>
<script>
import VueDRR from './vue-drag-resize-rotate-updater'
export default {
  name: 'app',
  components: {
    'vue-drr': VueDRR
  },
  data () {
    return {
      currentUid: ''
    }
  },
  methods: {
    showchange: function (data) {
      this.currentUid = data.uid
    }
  }
}
</script>
