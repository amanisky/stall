<template>
<div id="app">
  <el-container>
    <el-header>装修展位</el-header>
    <el-container>
      <el-aside width="300px">
        <el-collapse v-model="activeNames">
          <el-collapse-item title="通用" name="1">
            <div class="myicon">
              <img src="@/images/deng.svg" alt="灯">
              <img src="@/images/guizi.svg" alt="柜子">
            </div>
          </el-collapse-item>
          <el-collapse-item title="图标" name="2">
            <div class="myicon">
              <img src="@/images/deng.svg" alt="灯">
              <img src="@/images/guizi.svg" alt="柜子">
            </div>
          </el-collapse-item>
        </el-collapse>
      </el-aside>
      <el-main>
        <div id="canvas-box">
          <canvas ref="c"></canvas>
        </div>
      </el-main>
      <el-aside width="200px">
        111222
      </el-aside>
    </el-container>
  </el-container>
</div>
</template>

<script>
import { fabric } from 'fabric'
import stall from '@/images/stall.svg'
export default {
  name: 'App',
  data () {
    return {
      canvas: null,
      activeNames: [1]
    }
  },
  mounted () {
    fabric.Group.prototype.hasControls = false
    fabric.Group.prototype.lockMovementX = true
    fabric.Group.prototype.lockMovementY = true
    var canvasBox = document.getElementById('canvas-box')
    this.canvas = new fabric.Canvas(this.$refs.c, {
      width: canvasBox.clientWidth,
      height: canvasBox.clientHeight,
      backgroundColor: '#ccc'
    })
    fabric.loadSVGFromURL(stall, (objs, opts) => {
      var obj = fabric.util.groupSVGElements(objs, opts)
      for (var i = 0; i < 4; i++) {
        obj.clone((clone) => {
          clone.scale(2)
          var text = new fabric.Text(`B${40 + i}`, { top: clone.get('height') * 2 })
          text.set('left', (clone.get('width') * 2 - text.get('width')) / 2)
          var group = new fabric.Group([clone, text], { left: clone.get('width') * 2 * i + 2, borderColor: 'red' })
          this.canvas.add(group)
        })
      }
    })
  }
}
</script>

<style>
.el-header, .el-footer {
  background-color: #B3C0D1;
  color: #333;
  text-align: center;
  line-height: 60px;
}

.el-aside {
  height: calc(100vh - 60px);
  color: #333;
  text-align: center;
  background-color: #D3DCE6;
  overflow-y: auto;
  overflow-x: hidden;
}

.el-main {
  height: calc(100vh - 60px);
  color: #333;
  background-color: #E9EEF3;
}

#canvas-box {
  height: 100%;
}

.myicon {
  display: flex;
}

.myicon img {
  width: 30px;
  height: 30px;
}
</style>
