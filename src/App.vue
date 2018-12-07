<template>
<div id="app">
  <el-container class="draw">
    <el-aside width="200px">
      <el-collapse v-model="activeNames">
        <el-collapse-item title="通用" name="1">
          <div class="myicon">
            <img @click="addToCanvas('deng')" src="@/images/deng.svg" alt="灯">
            <img @click="addToCanvas('guizi')" src="@/images/guizi.svg" alt="柜子">
          </div>
        </el-collapse-item>
        <el-collapse-item title="图标" name="2">
          <div class="myicon">
            <img @click="addToCanvas('deng')" src="@/images/deng.svg" alt="灯">
            <img @click="addToCanvas('guizi')" src="@/images/guizi.svg" alt="柜子">
          </div>
        </el-collapse-item>
      </el-collapse>
    </el-aside>
    <el-container>
      <el-header>
        <el-button-group>
          <el-button size="small" type="danger">删除选中</el-button>
          <el-button size="small" type="danger">打通展位</el-button>
          <el-button size="small" type="danger">恢复默认</el-button>
        </el-button-group>
        <el-button-group>
          <el-button size="small" type="primary">左旋转</el-button>
          <el-button size="small" type="primary">右旋转</el-button>
          <el-button size="small" type="primary">调层距</el-button>
        </el-button-group>
        <el-button-group>
          <el-button size="small">撤销</el-button>
          <el-button size="small">重做</el-button>
        </el-button-group>
        <el-button size="small" type="success">保存</el-button>
        <el-button size="small" type="info">返回</el-button>
      </el-header>
      <el-main>
        <div id="canvas-box">
          <canvas ref="c"></canvas>
        </div>
      </el-main>
    </el-container>
  </el-container>
</div>
</template>

<script>
import { fabric } from 'fabric'
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
    fabric.Image.fromURL(this.requireSVG('stall'), (img) => {
      img.scale(2)
      var width = img.getScaledWidth()
      var height = img.getScaledHeight()
      for (let i = 0; i < 3; i++) {
        img.clone(obj => {
          var text = new fabric.Text(`B${40 + i}`, { top: height })
          text.set('left', (width - text.width) / 2)
          var group = new fabric.Group([obj, text], { left: width * i + 2, borderColor: 'red' })
          this.canvas.add(group)
        })
      }
    })
  },
  methods: {
    addToCanvas (url) {
      fabric.Image.fromURL(this.requireSVG(url), (img) => {
        img.scale(0.3)
        this.canvas.add(img)
      })
    },
    requireSVG (name) {
      return require(`@/images/${name}.svg`)
    }
  }
}
</script>

<style scoped>
.draw .el-header {
  height: 50px !important;
  color: #333;
  padding-top: 9px;
  background-color: #B3C0D1;
}

.draw .el-aside {
  height: 100vh;
  color: #333;
  text-align: center;
  background-color: #D3DCE6;
  overflow-y: auto;
  overflow-x: hidden;
}

.draw .el-main {
  height: calc(100vh - 50px);
  color: #333;
  background-color: #E9EEF3;
}

#canvas-box {
  height: 100%;
}

.myicon {
  display: flex;
  flex-wrap: wrap;
}

.myicon img {
  width: 30px;
  height: 30px;
  margin-right: 10px;
  cursor: pointer;
}
</style>
