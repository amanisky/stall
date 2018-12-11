<template>
<div id="app">
  <el-container class="draw">
    <el-aside width="200px">
      <el-collapse v-model="activeNames">
        <el-collapse-item title="用电类展具" name="1">
          <div>
            长臂射灯、金卤灯、日光灯、电箱、插座
          </div>
          <div class="myicon">
            <img @click="addShape('deng')" src="@/images/deng.svg" alt="灯">
            <img @click="addShape('guizi')" src="@/images/guizi.svg" alt="柜子">
          </div>
        </el-collapse-item>
        <el-collapse-item title="装搭类展具" name="2">
          <div>
            网片、地毯、围板、立柱、楣板、搁板、冲孔板、槽板、展柜、展架、地柜、饰柜、高低柜、报到台
          </div>
          <div class="myicon">
            <img @click="addShape('deng')" src="@/images/deng.svg" alt="灯">
            <img @click="addShape('guizi')" src="@/images/guizi.svg" alt="柜子">
          </div>
        </el-collapse-item>
        <el-collapse-item title="洽谈类展具" name="3">
          <div>
            洽谈桌、洽谈椅
          </div>
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
          <!-- <el-button size="small" type="danger" @click="reset">恢复默认</el-button> -->
        </el-button-group>
        <el-button-group>
          <el-button size="small" type="primary" @click="leftRotate">左旋转</el-button>
          <el-button size="small" type="primary" @click="rightRotate">右旋转</el-button>
          <el-button size="small" type="primary">调层距</el-button>
        </el-button-group>
        <el-button-group>
          <el-button size="small" type="info" :disabled="undoDisabled" @click="undo">撤销</el-button>
          <el-button size="small" type="info" :disabled="redoDisabled" @click="redo">重做</el-button>
        </el-button-group>
        <el-button size="small" type="success" @click="save">保存</el-button>
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
      // 画布初始状态
      canvasInitState: null,
      // 当前画布状态
      canvasState: [],
      // 当前状态索引
      currentStateIndex: -1,
      // undo 按钮状态
      undoStatus: false,
      // undo 是否禁用
      undoDisabled: true,
      // undo 按钮完成状态
      undoFinishedStatus: true,
      // redo 按钮状态
      redoStatus: false,
      // redo 是否禁用
      redoDisabled: true,
      // redo 按钮完成状态
      redoFinishedStatus: true
    }
  },
  mounted () {
    fabric.Object.prototype.hasControls = false
    fabric.Group.prototype.hoverCursor = 'default'
    fabric.Group.prototype.selectable = false
    this.initCanvas()
  },
  methods: {
    // 初始化画布
    initCanvas () {
      var canvasBox = document.getElementById('canvas-box')
      this.canvas = new fabric.Canvas(this.$refs.c, {
        width: canvasBox.clientWidth,
        height: canvasBox.clientHeight,
        backgroundColor: '#fff'
      })
      this.canvas
        .on('object:modified', () => {
          this.updateCanvasState()
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
    // 添加形状
    addShape (url) {
      fabric.Image.fromURL(this.requireSVG(url), (img) => {
        img.scale(0.3)
        this.canvas.add(img).bringToFront(img).setActiveObject(img)
        this.updateCanvasState()
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
        this.updateCanvasState()
      }
    },
    // 打通展位
    through () {
      var rect = new fabric.Rect({ top: 1.5, left: this.canvas.item(0).item(0).getScaledWidth() - 15, width: 30, height: this.canvas.item(0).item(0).getScaledHeight() - 4, fill: '#fff', lockMovementY: true })
      this.canvas.add(rect).setActiveObject(rect)
      this.updateCanvasState()
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
    },
    // 更新画布状态
    updateCanvasState () {
      if ((this.undoStatus === false && this.redoStatus === false)) {
        var jsonData = this.canvas.toJSON()
        var canvasAsJson = JSON.stringify(jsonData)
        if (this.currentStateIndex < this.canvasState.length - 1) {
          // 插入状态的索引值
          var indexToBeInserted = this.currentStateIndex + 1
          this.canvasState[indexToBeInserted] = canvasAsJson
          // 保留的元素数量
          var numberOfElementsToRetain = indexToBeInserted + 1
          this.canvasState = this.canvasState.splice(0, numberOfElementsToRetain)
        } else {
          this.canvasState.push(canvasAsJson)
          this.undoDisabled = false
        }
        this.currentStateIndex = this.canvasState.length - 1
        if ((this.currentStateIndex === this.canvasState.length - 1) && this.currentStateIndex !== -1) {
          this.redoDisabled = true
        }
      }
    },
    // 撤销
    undo () {
      if (this.undoFinishedStatus) {
        if (this.currentStateIndex === -1) {
          this.undoStatus = false
        } else {
          if (this.canvasState.length >= 1) {
            this.undoFinishedStatus = false
            if (this.currentStateIndex !== 0) {
              this.undoStatus = true
              this.canvas.loadFromJSON(this.canvasState[this.currentStateIndex - 1], () => {
                this.canvas.renderAll()
                this.undoStatus = false
                this.currentStateIndex -= 1
                this.undoDisabled = false
                if (this.currentStateIndex !== this.canvasState.length - 1) {
                  this.redoDisabled = false
                }
                this.undoFinishedStatus = true
              })
            } else if (this.currentStateIndex === 0) {
              // this.canvas.clear()
              this.canvas.loadFromJSON(this.canvasInitState, this.canvas.renderAll.bind(this.canvas))
              this.undoFinishedStatus = true
              this.undoDisabled = true
              this.redoDisabled = false
              this.currentStateIndex -= 1
            }
          }
        }
      }
    },
    // 重做
    redo () {
      if (this.redoFinishedStatus) {
        if ((this.currentStateIndex === this.canvasState.length - 1) && this.currentStateIndex !== -1) {
          this.redoDisabled = true
        } else {
          if (this.canvasState.length > this.currentStateIndex && this.canvasState.length !== 0) {
            this.redoFinishedStatus = false
            this.redoStatus = true
            this.canvas.loadFromJSON(this.canvasState[this.currentStateIndex + 1], () => {
              this.canvas.renderAll()
              this.redoStatus = false
              this.currentStateIndex += 1
              if (this.currentStateIndex !== -1) {
                this.undoDisabled = false
              }
              this.redoFinishedStatus = true
              if ((this.currentStateIndex === this.canvasState.length - 1) && this.currentStateIndex !== -1) {
                this.redoDisabled = true
              }
            })
          }
        }
      }
    },
    // 保存
    save () {
      // console.log(this.canvas.toJSON())
      console.log(this.canvasState.length)
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
