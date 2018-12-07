<template>
<div id="app">
  <el-container class="draw">
    <el-aside width="200px">
      <el-collapse v-model="activeNames">
        <el-collapse-item title="通用" name="1">
          <div class="myicon">
            <img @click="addShape('deng')" src="@/images/deng.svg" alt="灯">
            <img @click="addShape('guizi')" src="@/images/guizi.svg" alt="柜子">
          </div>
        </el-collapse-item>
        <el-collapse-item title="图标" name="2">
          <div class="myicon">
            <img @click="addShape('deng')" src="@/images/deng.svg" alt="灯">
            <img @click="addShape('guizi')" src="@/images/guizi.svg" alt="柜子">
          </div>
        </el-collapse-item>
      </el-collapse>
    </el-aside>
    <el-container>
      <el-header>
        <el-button-group>
          <el-button size="small" type="danger" @click="removeSelected">删除选中</el-button>
          <el-button size="small" type="danger" @click="through">打通展位</el-button>
          <el-button size="small" type="danger" @click="reset">恢复默认</el-button>
        </el-button-group>
        <el-button-group>
          <el-button size="small" type="primary" @click="leftRotate">左旋转</el-button>
          <el-button size="small" type="primary" @click="rightRotate">右旋转</el-button>
          <el-button size="small" type="primary">调层距</el-button>
        </el-button-group>
        <el-button-group>
          <el-button size="small" type="info">撤销</el-button>
          <el-button size="small" type="info">重做</el-button>
        </el-button-group>
        <el-button size="small" type="success">保存</el-button>
        <el-button size="small">返回</el-button>
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
      activeNames: [1],
      canvasState: [],
      canvasInitState: null
    }
  },
  mounted () {
    fabric.Object.prototype.hasControls = false
    fabric.Group.prototype.hoverCursor = 'default'
    fabric.Group.prototype.selectable = false
    var canvasBox = document.getElementById('canvas-box')
    this.canvas = new fabric.Canvas(this.$refs.c, {
      width: canvasBox.clientWidth,
      height: canvasBox.clientHeight,
      backgroundColor: '#fff'
    })
    fabric.Image.fromURL(this.requireSVG('stall'), (img) => {
      img.scale(2)
      var width = img.getScaledWidth()
      var height = img.getScaledHeight()
      for (let i = 0; i < 4; i++) {
        img.clone(obj => {
          var text = new fabric.Text(`B${40 + i}`, { top: height })
          text.set('left', (width - text.width) / 2)
          var group = new fabric.Group([obj, text], { left: width * i + 2, lockMovementX: true, lockMovementY: true })
          this.canvas.add(group)
          this.canvasInitState = this.canvas.toJSON()
        })
      }
    })
  },
  methods: {
    // 添加形状
    addShape (url) {
      fabric.Image.fromURL(this.requireSVG(url), (img) => {
        img.scale(0.3)
        this.canvas.add(img).bringToFront(img).setActiveObject(img)
      })
    },
    // 获取SVG
    requireSVG (name) {
      return require(`@/images/${name}.svg`)
    },
    // 删除选中
    removeSelected () {
      var activeObjects = this.canvas.getActiveObjects()
      this.canvas.discardActiveObject()
      if (activeObjects.length) {
        this.canvas.remove.apply(this.canvas, activeObjects)
      }
    },
    // 打通展位
    through () {
      var rect = new fabric.Rect({ top: 1.5, left: this.canvas.item(0).item(0).getScaledWidth() - 15, width: 30, height: this.canvas.item(0).item(0).getScaledHeight() - 4, fill: '#fff', lockMovementY: true })
      this.canvas.add(rect).setActiveObject(rect)
    },
    // 恢复默认
    reset () {
      this.canvas.loadFromJSON(this.canvasInitState, this.canvas.renderAll.bind(this.canvas))
    },
    // 左旋转
    leftRotate () {
      var activeObject = this.canvas.getActiveObject()
      var angle = activeObject.angle
      activeObject.rotate(angle - 90).setCoords()
      this.canvas.requestRenderAll()
    },
    // 右旋转
    rightRotate () {
      var activeObject = this.canvas.getActiveObject()
      var angle = activeObject.angle
      activeObject.rotate(angle + 90).setCoords()
      this.canvas.requestRenderAll()
    }
  }
}
</script>

<style scoped>
.draw .el-header {
  height: 50px !important;
  color: #333;
  padding-top: 9px;
  background-color: #b3c0d1;
}

.draw .el-aside {
  height: 100vh;
  color: #333;
  text-align: center;
  background-color: #c3cce6;
  overflow-y: auto;
  overflow-x: hidden;
}

.draw .el-main {
  height: calc(100vh - 50px);
  color: #333;
  background-color: #e9eef3;
}

.draw .el-button-group {
  margin-right: 10px;
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
