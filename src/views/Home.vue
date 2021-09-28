<template>
  <div class="home">
    <div class="top-area">
      <div class="score">{{ line }}</div>
      <div class="screen">
        <canvas class="hold-block" ref="holdBlockCanvas" :width="unitSize * 4" :height="unitSize * 4"></canvas>
        <!--   20 row  10 col  23px/格   -->
        <canvas class="canvas" ref="screenCanvas" :width="width" :height="height"></canvas>
      </div>
      <div class="top-right">
        <div class="current"></div>
        <div class="after"></div>
      </div>
    </div>
    <div class="bottom-area">
      <div class="cross-button">
        <div class="cross-button-item top" @click="changeDirection(0)"/>
        <div class="cross-button-item right" @click="changeDirection(1)"/>
        <div class="cross-button-item bottom" @click="changeDirection(2)"/>
        <div class="cross-button-item left" @click="changeDirection(3)"/>
      </div>
      <div class="function-button">
        <div class="round-button button-a">A</div>
        <div class="round-button button-b">B</div>
        <div class="round-button button-c">X</div>
        <div class="round-button button-d">Y</div>
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
const blockArray = ['T', 'I', 'Z', 'ReZ', 'L', 'ReL', 'Tian'];
export default {
  name: 'Home',
  components: {},
  data() {
    return {
      screen: null,
      screenCtx: null,
      holdCtx: null,
      unitSize: 23,
      width: 230,
      height: 460,
      usedBlock: [],    // 已经使用过的
      notUseBlock: [],  // 未使用的
      gameIsEnd: false,
      currentBlock: {name: '', x: 0, y: 0, directionIndex: 0},
      nextBlock: {name: '', x: 0, y: 0, directionIndex: 0},
      holdBlock: {name: '', x: 0, y: 0, directionIndex: 0},
      futureBlockArray: [], // 3 块
      speed: 1000,   // 速度
      frameTimeout: null,
      contactStartTime: false,  // 碰撞开始时间
      directionArray: ['up', 'right', 'down', 'left'],
      lastImageData: false,   // 上一次发生碰撞时的游戏画面。
      line: 0,    // 消除的行数
    }
  },
  mounted() {
    this.init()
    this.addKeyboardListener();
  },
  beforeUnmount() {
    this.frameTimeout && clearTimeout(this.frameTimeout)
    this.removeKeyboardListener()
  },
  methods: {
    init() {
      this.drawLine()
      this.randomList()
      this.run()
    },
    changeDirection(direction) {
      let doList = {
        0: () => {
        },
        1: () => {
        },
        2: () => {
        },
        3: () => {
        },
      }
      doList[direction] && doList[direction]();
    },
    // 在canvas中画出网格线
    drawLine() {
      this.screen = this.$refs.screenCanvas
      let screenCtx = this.screen.getContext('2d')
      this.holdCtx = this.$refs.holdBlockCanvas.getContext('2d')
      this.screenCtx = screenCtx;
      screenCtx.strokeStyle = '#f00'
      for (let i = 0; i < 10; i++) {
        for (let j = 0; j < 20; j++) {
          screenCtx.strokeRect(i * this.unitSize, j * this.unitSize, this.unitSize, this.unitSize);
        }
      }
    },
    // 随机序列  保证7个一组  随机生成一个方块
    randomList() {
      if (this.notUseBlock.length === 0) {
        this.notUseBlock = blockArray;
        this.usedBlock = [];
      }
      // 随机生成
      let x = Math.floor(Math.random() * this.notUseBlock.length)
      let block = this.notUseBlock[x]
      this.usedBlock.push(block)
      this.notUseBlock = this.notUseBlock.filter(b => b !== block);
      this.currentBlock = {
        name: block,
        x: this.unitSize * 4,
        y: 0,
        directionIndex: 0
      };
      this['draw' + block](this.unitSize * 4, 0);
    },
    // T
    drawT(x, y) {
      x = x || this.currentBlock.x
      y = y || this.currentBlock.y
      let bottomY = y
      while (!this.collisionDetection()) {
        this.currentBlock.y += this.unitSize
        bottomY = this.currentBlock.y
      }
      this.currentBlock.y = y;

      // * x * *
      // x x x *
      // * * * *
      this.screenCtx.fillStyle = "#2fff00"
      this.screenCtx.strokeStyle = "#2fff00"
      let {directionIndex} = this.currentBlock
      if (directionIndex === 0) {
        // 如果超出了画布， 那就制止
        if (x >= 7 * this.unitSize) x = 7 * this.unitSize;
        if (x <= 0) x = 0;
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 1, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY + this.unitSize, this.unitSize, this.unitSize);
      } else if (directionIndex === 2) {
        if (x >= 7 * this.unitSize) x = 7 * this.unitSize;
        if (x <= 0) x = 0;
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 2, this.unitSize, this.unitSize);


        this.screenCtx.strokeRect(x, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize * 2, this.unitSize, this.unitSize);
      } else if (directionIndex === 3) {
        if (x >= 8 * this.unitSize) x = 8 * this.unitSize;
        if (x <= 0) x = 0;
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 2, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize * 2, this.unitSize, this.unitSize);
      } else if (directionIndex === 1) {
        if (x >= 7 * this.unitSize) x = 7 * this.unitSize;
        if (x <= -this.unitSize) x = -this.unitSize;
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 2, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize * 2, this.unitSize, this.unitSize);
      }
      this.currentBlock.x = x
    },
    // 棍
    drawI(x, y) {
      this.screenCtx.fillStyle = "#00ffc2"
      this.screenCtx.strokeStyle = "#00ffc2"
      x = x || this.currentBlock.x
      y = y || this.currentBlock.y
      let bottomY = y
      while (!this.collisionDetection()) {
        this.currentBlock.y += this.unitSize
        bottomY = this.currentBlock.y
      }
      this.currentBlock.y = y;

      // x x x x
      // * * * *
      let {directionIndex} = this.currentBlock
      if (directionIndex === 1) {
        if (x >= 7 * this.unitSize) x = 7 * this.unitSize;
        if (x <= -2 * this.unitSize) x = -2 * this.unitSize;
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 2, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 3, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY + this.unitSize * 2, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY + this.unitSize * 3, this.unitSize, this.unitSize);
      } else if (directionIndex === 3) {
        if (x >= 8 * this.unitSize) x = 8 * this.unitSize;
        if (x <= -this.unitSize) x = -this.unitSize;
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 2, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 3, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize * 2, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize * 3, this.unitSize, this.unitSize);
      } else {
        if (x >= 6 * this.unitSize) x = 6 * this.unitSize;
        if (x <= 0) x = 0;
        this.screenCtx.fillRect(x + this.unitSize * 0, y + +this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + +this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + +this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 3, y + +this.unitSize * 0, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 3, bottomY, this.unitSize, this.unitSize);
      }
      this.currentBlock.x = x
    },
    // Z
    drawZ(x, y) {
      this.screenCtx.fillStyle = "#0052ff"
      this.screenCtx.strokeStyle = "#0052ff"
      x = x || this.currentBlock.x
      y = y || this.currentBlock.y
      let bottomY = y
      while (!this.collisionDetection()) {
        this.currentBlock.y += this.unitSize
        bottomY = this.currentBlock.y
      }
      this.currentBlock.y = y;
      // x x * *
      // * x x *
      let {directionIndex} = this.currentBlock
      if (directionIndex === 0 || directionIndex === 2) {
        if (x >= 7 * this.unitSize) x = 7 * this.unitSize;
        if (x <= 0) x = 0;
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 1, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY + this.unitSize, this.unitSize, this.unitSize);
      } else if (directionIndex === 1 || directionIndex === 3) {
        if (x >= 8 * this.unitSize) x = 8 * this.unitSize;
        if (x <= 0) x = 0;
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 2, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x, bottomY + this.unitSize * 2, this.unitSize, this.unitSize);
      }
      this.currentBlock.x = x
    },
    // 反Z
    drawReZ(x, y) {
      this.screenCtx.fillStyle = "#c700ff"
      this.screenCtx.strokeStyle = "#c700ff"
      x = x || this.currentBlock.x
      y = y || this.currentBlock.y
      let bottomY = y
      while (!this.collisionDetection()) {
        this.currentBlock.y += this.unitSize
        bottomY = this.currentBlock.y
      }
      this.currentBlock.y = y;
      // * x x *
      // x x * *
      let {directionIndex} = this.currentBlock
      if (directionIndex === 0 || directionIndex === 2) {
        if (x >= 7 * this.unitSize) x = 7 * this.unitSize;
        if (x <= 0) x = 0;
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 0, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY, this.unitSize, this.unitSize);
      } else if (directionIndex === 1 || directionIndex === 3) {
        if (x >= 8 * this.unitSize) x = 8 * this.unitSize;
        if (x <= 0) x = 0;
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 2, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize * 2, this.unitSize, this.unitSize);
      }
      this.currentBlock.x = x
    },
    // L
    drawL(x, y) {
      this.screenCtx.fillStyle = "#ff006a"
      this.screenCtx.strokeStyle = "#ff006a"
      x = x || this.currentBlock.x
      y = y || this.currentBlock.y
      let bottomY = y
      while (!this.collisionDetection()) {
        this.currentBlock.y += this.unitSize
        bottomY = this.currentBlock.y
      }
      this.currentBlock.y = y;
      // * * x *
      // x x x *
      let {directionIndex} = this.currentBlock
      if (directionIndex === 0) {
        if (x <= 0) x = 0;
        if (x >= 7 * this.unitSize) x = 7 * this.unitSize;
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 0, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY, this.unitSize, this.unitSize);
      } else if (directionIndex === 1) {
        if (x <= 0) x = 0;
        if (x >= 8 * this.unitSize) x = 8 * this.unitSize;
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 2, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 2, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x, bottomY + this.unitSize * 2, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize * 2, this.unitSize, this.unitSize);
      } else if (directionIndex === 2) {
        if (x <= 0) x = 0;
        if (x >= 7 * this.unitSize) x = 7 * this.unitSize;
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x, bottomY + this.unitSize, this.unitSize, this.unitSize);
      } else if (directionIndex === 3) {
        if (x <= 0) x = 0;
        if (x >= 8 * this.unitSize) x = 8 * this.unitSize;
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 2, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize * 2, this.unitSize, this.unitSize);
      }
      this.currentBlock.x = x
    },
    // 反 L
    drawReL(x, y) {
      this.screenCtx.fillStyle = "#ff0000"
      this.screenCtx.strokeStyle = "#ff0000"
      x = x || this.currentBlock.x
      y = y || this.currentBlock.y
      let bottomY = y
      while (!this.collisionDetection()) {
        this.currentBlock.y += this.unitSize
        bottomY = this.currentBlock.y
      }
      this.currentBlock.y = y;
      // x * * *
      // x x x *

      let {directionIndex} = this.currentBlock
      if (directionIndex === 0) {
        if (x <= 0) x = 0;
        if (x >= 7 * this.unitSize) x = 7 * this.unitSize;
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 0, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x, bottomY, this.unitSize, this.unitSize);
      } else if (directionIndex === 1) {
        if (x >= 8 * this.unitSize) x = 8 * this.unitSize;
        if (x <= 0) x = 0;
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 2, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x, bottomY + this.unitSize * 2, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
      } else if (directionIndex === 2) {
        if (x <= 0) x = 0;
        if (x >= 7 * this.unitSize) x = 7 * this.unitSize;
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 1, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize * 2, bottomY + this.unitSize, this.unitSize, this.unitSize);
      } else if (directionIndex === 3) {
        if (x <= 0) x = 0;
        if (x >= 8 * this.unitSize) x = 8 * this.unitSize;
        this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 2, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
        this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 2, this.unitSize, this.unitSize);

        this.screenCtx.strokeRect(x, bottomY + this.unitSize * 2, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
        this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize * 2, this.unitSize, this.unitSize);
      }
      this.currentBlock.x = x
    },
    // 田
    drawTian(x, y) {
      this.screenCtx.fillStyle = "#ff6a00"
      this.screenCtx.strokeStyle = "#ff6a00"
      x = x || this.currentBlock.x
      y = y || this.currentBlock.y
      let bottomY = y
      while (!this.collisionDetection()) {
        this.currentBlock.y += this.unitSize
        bottomY = this.currentBlock.y
      }
      this.currentBlock.y = y;
      // x x * *
      // x x * *
      if (x >= 8 * this.unitSize) x = 8 * this.unitSize;
      if (x <= 0) x = 0;
      this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 0, this.unitSize, this.unitSize);
      this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
      this.screenCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
      this.screenCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);

      this.screenCtx.strokeRect(x, bottomY, this.unitSize, this.unitSize);
      this.screenCtx.strokeRect(x + this.unitSize, bottomY, this.unitSize, this.unitSize);
      this.screenCtx.strokeRect(x, bottomY + this.unitSize, this.unitSize, this.unitSize);
      this.screenCtx.strokeRect(x + this.unitSize, bottomY + this.unitSize, this.unitSize, this.unitSize);
      this.currentBlock.x = x
    },
    // T
    holdDrawT(x, y) {
      x = x || this.holdBlock.x
      y = y || this.holdBlock.y

      // * x * *
      // x x x *
      // * * * *
      this.holdCtx.fillStyle = "#2fff00"
      this.holdCtx.strokeStyle = "#2fff00"

      this.holdCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 1, this.unitSize, this.unitSize);
    },
    // 棍
    holdDrawI(x, y) {
      this.holdCtx.fillStyle = "#00ffc2"
      this.holdCtx.strokeStyle = "#00ffc2"
      x = x || this.holdBlock.x
      y = y || this.holdBlock.y
      this.holdCtx.fillRect(x + this.unitSize * 0, y + +this.unitSize * 0, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 1, y + +this.unitSize * 0, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 2, y + +this.unitSize * 0, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 3, y + +this.unitSize * 0, this.unitSize, this.unitSize);
    },
    // Z
    holdDrawZ(x, y) {
      this.holdCtx.fillStyle = "#0052ff"
      this.holdCtx.strokeStyle = "#0052ff"
      x = x || this.holdBlock.x
      y = y || this.holdBlock.y
      // x x * *
      // * x x *
      this.holdCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 0, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 1, this.unitSize, this.unitSize);
    },
    // 反Z
    holdDrawReZ(x, y) {
      this.holdCtx.fillStyle = "#c700ff"
      this.holdCtx.strokeStyle = "#c700ff"
      x = x || this.holdBlock.x
      y = y || this.holdBlock.y
      // * x x *
      // x x * *
      this.holdCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 0, this.unitSize, this.unitSize);
    },
    // L
    holdDrawL(x, y) {
      this.holdCtx.fillStyle = "#ff006a"
      this.holdCtx.strokeStyle = "#ff006a"
      x = x || this.holdBlock.x
      y = y || this.holdBlock.y
      // * * x *
      // x x x *
      this.holdCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 1, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 0, this.unitSize, this.unitSize);
    },
    // 反 L
    holdDrawReL(x, y) {
      this.holdCtx.fillStyle = "#ff0000"
      this.holdCtx.strokeStyle = "#ff0000"
      x = x || this.holdBlock.x
      y = y || this.holdBlock.y
      // x * * *
      // x x x *

      this.holdCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 2, y + this.unitSize * 1, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 0, this.unitSize, this.unitSize);
    },
    // 田
    holdDrawTian(x, y) {
      this.holdCtx.fillStyle = "#ff6a00"
      this.holdCtx.strokeStyle = "#ff6a00"
      x = x || this.holdBlock.x
      y = y || this.holdBlock.y

      this.holdCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 0, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 0, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 0, y + this.unitSize * 1, this.unitSize, this.unitSize);
      this.holdCtx.fillRect(x + this.unitSize * 1, y + this.unitSize * 1, this.unitSize, this.unitSize);

    },
    // 运动 更新每一帧
    run() {
      let self = this
      requestAnimationFrame(function aFrame() {
        self.frameTimeout && clearTimeout(self.frameTimeout)
        self.frameTimeout = setTimeout(() => {
          if (!self.gameIsEnd) {
            requestAnimationFrame(aFrame)
          }
        }, self.speed)

        // 如果发生碰撞，y就不增加
        if (!self.contactStartTime) {
          self.currentBlock.y += self.unitSize
        }

        self.updateCanvas();

        // 如果碰撞时间超过了最大时间， 就切换方块
        let currentTime = new Date().getTime();
        if (self.contactStartTime && (currentTime - self.contactStartTime >= self.speed)) {
          // 消行检测
          self.line += self.removeLine()
          self.contactStartTime = false;
          self.lastImageData = self.screenCtx.getImageData(0, 0, self.width, self.height);
          // 发生碰撞时  最上面中间两块是否被填充
          self.checkGameIsEnd();
          // 检测游戏是否结束
          self.randomList();
        }

        // 移动之后 进行碰撞检测
        self.collisionDetection();
      })
    },
    // 纵向碰撞检测  检测 currentBlock 和 底部的方块是否有碰撞
    // 有碰撞， 就停下，判断游戏是否结束， 若没结束 则 生成新的方块，
    collisionDetection() {
      // 游戏区域  x 10 * this.unitSize  y 20 * this.unitSize
      // 需要知道底边的y  根据方向来

      // 黑色 0 0 0 255
      // get 方块的底边 + 1行 然后判断底边的下面那一条边 是否颜色都是黑的。

      let {name, directionIndex, x, y} = this.currentBlock
      let contactFlag = false;
      if (name === 'T') {
        let imageData = this.screenCtx.getImageData(x, y, this.unitSize * 3, this.unitSize * 4)
        if (directionIndex === 0) {
          // * x * *
          // x x x *
          // ? ? ? -
          if ((imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 18)) {
            contactFlag = true;
          }
        } else if (directionIndex === 3) {
          // * x * *   * x * *
          // x x * *   * x x *
          // ? x * *   * x ? *
          // - ? - -   - ? - -
          if ((imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 17)) {
            contactFlag = true;
          }
        } else if (directionIndex === 1) {
          if ((imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 17)) {
            contactFlag = true;
          }
        } else if (directionIndex === 2) {
          // * * * *
          // x x x *
          // ? x ? *
          // - ? - -
          if ((imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 18)) {
            contactFlag = true;
          }
        }
      } else if (name === 'I') {
        let imageData = this.screenCtx.getImageData(x, y, this.unitSize * 4, this.unitSize * 5)
        if (directionIndex === 3) {
          // * x * *  left right  * * x *
          // * x * *              * * x *
          // * x * *              * * x *
          // * x * *              * * x *
          // * ? * *              * * ? *
          //                          行                                           列
          if ((imageData.data[((4 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 16)) {
            contactFlag = true;
          }
        } else if (directionIndex === 1) {
          if ((imageData.data[((4 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 16)) {
            contactFlag = true;
          }
        } else {
          // x x x x
          // ? ? ? ?
          if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 19)) {
            contactFlag = true;
          }
        }
      } else if (name === 'Z') {
        // up             right         down            left
        // x x * *        * x * *       x x * *         * x * *
        // ? x x *        x x * *       ? x x *         x x * *
        // * ? ? *        x ? * *       * ? ? *         x ? * *
        //                ?                             ?
        let imageData = this.screenCtx.getImageData(x, y, this.unitSize * 3, this.unitSize * 4)
        if (directionIndex === 0 || directionIndex === 2) {
          if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 18)) {
            contactFlag = true;
          }
        } else {
          if ((imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              ((y >= this.unitSize * 17))) {
            contactFlag = true;
          }
        }
      } else if (name === 'ReZ') {
        // up             right
        // * x x *        x * * *
        // x x ? *        x x * *
        // ? ? * *        ? x * *
        //                  ?
        let imageData = this.screenCtx.getImageData(x, y, this.unitSize * 3, this.unitSize * 4)
        if (directionIndex === 0 || directionIndex === 2) {
          if ((imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 18)) {
            contactFlag = true;
          }
        } else {
          if ((imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 17)) {
            contactFlag = true;
          }
        }
      } else if (name === 'L') {
        // up             right         down            left
        // * * x *        x * * *       x x x *         x x * *
        // x x x *        x * * *       x ? ? *         ? x * *
        // ? ? ? *        x x * *       ? * * *         * x * *
        //                ? ?                           * ?
        let imageData = this.screenCtx.getImageData(x, y, this.unitSize * 3, this.unitSize * 4);
        if (directionIndex === 0) {
          if ((imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 18)) {
            contactFlag = true;
          }
        } else if (directionIndex === 2) {
          if ((imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 18)) {
            contactFlag = true;
          }
        } else if (directionIndex === 1) {
          if ((imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 17)) {
            contactFlag = true;
          }
        } else if (directionIndex === 3) {
          if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 17)) {
            contactFlag = true;
          }
        }
      } else if (name === 'ReL') {
        // up             right         down            left
        // x * * *        x x * *       x x x *         * x * *
        // x x x *        x ? * *       ? ? x *         * x * *
        // ? ? ? *        x * * *       * * ? *         x x * *
        //                ?                             ? ?
        let imageData = this.screenCtx.getImageData(x, y, this.unitSize * 3, this.unitSize * 4);
        if (directionIndex === 0) {
          if ((imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 18)) {
            contactFlag = true;
          }
        } else if (directionIndex === 2) {
          if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 18)) {
            contactFlag = true;
          }
        } else if (directionIndex === 3) {
          if ((imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 17)) {
            contactFlag = true;
          }
        } else if (directionIndex === 1) {
          if ((imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (y >= this.unitSize * 17)) {
            contactFlag = true;
          }
        }
      } else if (name === 'Tian') {
        // x x
        // x x
        // ? ?
        let imageData = this.screenCtx.getImageData(x, y, this.unitSize * 3, this.unitSize * 4);
        if ((imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
            (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
            (y >= this.unitSize * 18)) {
          contactFlag = true;
        }
      }
      // 如果发生了碰撞， 就开始计时  在run中计算， 如果超过最大碰撞时间就   切换方块

      if (contactFlag && !this.contactStartTime) {
        this.contactStartTime = new Date().getTime();
      } else if (!contactFlag) {
        this.contactStartTime = false
      }
      return contactFlag;
    },
    // 横向碰撞检测
    // 判断左边和右边有无色块
    rowCollisionDetection(key) {
      let {name, directionIndex, x, y} = this.currentBlock
      let contactFlag = false;
      if (name === 'T') {
        let imageData = this.screenCtx.getImageData(x - this.unitSize, y, this.unitSize * 5, this.unitSize * 3)
        if (directionIndex === 0) {
          //* ? x ? *
          //? x x x ?
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 3) {
          //* ? x ? *   * x * *
          //? x x ? *   * x x *
          //* ? x ? *   * x ? *
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 1) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 2) {
          // * * * * *
          // ? x x x ?
          // * ? x ? *
          if (key === 'left') {
            if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        }
      } else if (name === 'I') {
        let imageData = this.screenCtx.getImageData(x - this.unitSize, y, this.unitSize * 6, this.unitSize * 4)
        if (directionIndex === 3) {
          // * x * *  left right  * * x *
          // * x * *              * * x *
          // * x * *              * * x *
          // * x * *              * * x *
          // * ? * *              * * ? *
          //                          行                                           列
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 1) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else {
          //? x x x x ?
          //* * * * * *
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 5 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        }
      } else if (name === 'Z') {
        // up             right         down            left
        //? x x * *       * ? x * *       x x * *         * x * *
        //* ? x x *       ? x x * *       ? x x *         x x * *
        //* * * * *       ? x ? * *       * ? ? *         x ? * *
        //                ?                             ?
        let imageData = this.screenCtx.getImageData(x - this.unitSize, y, this.unitSize * 5, this.unitSize * 3)
        if (directionIndex === 0 || directionIndex === 2) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        }
      } else if (name === 'ReZ') {
        // up             right
        //* ? x x *        x * * *
        //? x x ? *        x x * *
        //* ? ? * *        ? x * *
        //                  ?
        let imageData = this.screenCtx.getImageData(x - this.unitSize, y, this.unitSize * 5, this.unitSize * 3)
        if (directionIndex === 0 || directionIndex === 2) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        }
      } else if (name === 'L') {
        // up             right         down            left
        // * * x *        x * * *       x x x *         x x * *
        // x x x *        x * * *       x ? ? *         ? x * *
        // ? ? ? *        x x * *       ? * * *         * x * *
        //                ? ?                           * ?
        let imageData = this.screenCtx.getImageData(x - this.unitSize, y, this.unitSize * 5, this.unitSize * 3);
        if (directionIndex === 0) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 2) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 1) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 3) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        }
      } else if (name === 'ReL') {
        // up             right         down            left
        // x * * *        x x * *       x x x *         * x * *
        // x x x *        x ? * *       ? ? x *         * x * *
        // ? ? ? *        x * * *       * * ? *         x x * *
        //                ?                             ? ?
        let imageData = this.screenCtx.getImageData(x - this.unitSize, y, this.unitSize * 5, this.unitSize * 3);
        if (directionIndex === 0) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 2) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 4 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 3) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 1) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        }
      } else if (name === 'Tian') {
        // x x
        // x x
        // ? ?
        let imageData = this.screenCtx.getImageData(x - this.unitSize, y, this.unitSize * 4, this.unitSize * 2);
        if (key === 'left') {
          if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
            contactFlag = true;
          }
        } else if (key === 'right') {
          if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255)) {
            contactFlag = true;
          }
        }
      }
      return contactFlag;
    },
    // 旋转碰撞检测
    // 左旋  判断附近一片的方块，可以就转， 不行就不给转
    // 当前的方向  和下一个方向   下一个方向与当前方向的差
    rotateCollisionDetection(key) {
      let {name, directionIndex, x, y} = this.currentBlock
      let contactFlag = false;


      if (name === 'T') {
        let imageData = this.screenCtx.getImageData(x, y, this.unitSize * 4, this.unitSize * 3)
        if (directionIndex === 0) {
          //* x * *
          //x x x *
          //* ? * *
          if ((imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
            contactFlag = true;
          }
        } else if (directionIndex === 3) {
          // * x * *   * x * *
          // x x ? *   ? x x *
          // * x * *   * x * *
          if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255)) {
            contactFlag = true;
          }
        } else if (directionIndex === 1) {
          if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
            contactFlag = true;
          }
        } else if (directionIndex === 2) {
          // * ? * *
          // x x x *
          // * x * *
          if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
            contactFlag = true;
          }
        }
      } else if (name === 'I') {
        let imageData = this.screenCtx.getImageData(x, y, this.unitSize * 6, this.unitSize * 4)
        if (directionIndex === 3) {
          // ? x ? ?  left right  * * x *
          // * x * *              * * x *
          // * x * *              * * x *
          // * x * *              * * x *
          // * ? * *              * * ? *
          //                          行                                           列
          if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255)) {
            contactFlag = true;
          }
        } else if (directionIndex === 1) {
          if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 3 * this.unitSize * 4) + 3] === 255)) {
            contactFlag = true;
          }
        } else if (directionIndex === 0) {
          //? x x x x ?
          //* ? * * * *
          //* ?
          //* ?
          if (key === 'left') {
            if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 2) {
          if (key === 'left') {
            if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((3 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        }
      } else if (name === 'Z') {
        // up             right         down            left
        // x x * *        ? x * *       x x * *         * x * *
        // ? x x *        x x ? *       ? x x *         x x * *
        // ? * * *        x * * *       * ? ? *         x ? * *
        //
        let imageData = this.screenCtx.getImageData(x, y, this.unitSize * 5, this.unitSize * 3)
        if (directionIndex === 0 || directionIndex === 2) {
          if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
            contactFlag = true;
          }
        } else {
          if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255)) {
            contactFlag = true;
          }
        }
      } else if (name === 'ReZ') {
        // up             right
        // ? x x *        x * * *
        // x x * *        x x * *
        // * ? * *        ? x * *
        //                  ?
        let imageData = this.screenCtx.getImageData(x, y, this.unitSize * 5, this.unitSize * 3)
        if (directionIndex === 0 || directionIndex === 2) {
          if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
            contactFlag = true;
          }
        } else {
          if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
              (imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255)) {
            contactFlag = true;
          }
        }
      } else if (name === 'L') {
        // up             right         down            left
        // ? ? x *        x * ? *       x x x *         x x ? *
        // x x x *        x ? ? *       x ? * *         ? x ? *
        // * ? * *        x x * *       ? ? * *         * x * *
        //                ? ?                           * ?
        let imageData = this.screenCtx.getImageData(x, y, this.unitSize * 5, this.unitSize * 3);
        if (directionIndex === 0) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 2) {
          if (key === 'left') {
            if ((imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 1) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 3) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        }
      } else if (name === 'ReL') {
        // up             right         down            left
        // x * * *        x x * *       x x x *         * x * *
        // x x x *        x ? * *       ? ? x *         * x * *
        // ? ? ? *        x * * *       * * ? *         x x * *
        //                ?                             ? ?
        let imageData = this.screenCtx.getImageData(x, y, this.unitSize * 5, this.unitSize * 3);
        if (directionIndex === 0) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 2) {
          if (key === 'left') {
            if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((2 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 1) {
          if (key === 'left') {
            if ((imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 1 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        } else if (directionIndex === 3) {
          if (key === 'left') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          } else if (key === 'right') {
            if ((imageData.data[((0 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 0 * this.unitSize * 4) + 3] === 255) ||
                (imageData.data[((1 * this.unitSize * (imageData.width * 4)) + 2 * this.unitSize * 4) + 3] === 255)) {
              contactFlag = true;
            }
          }
        }
      } else if (name === 'Tian') {
        // x x
        // x x
        contactFlag = false
      }
      return contactFlag;
    },
    // 监听键盘事件
    // 移动之后要重新碰撞检测
    addKeyboardListener() {
      document.addEventListener('keydown', (e) => {
        // e.key === w s a d ArrowUp ArrowDown ArrowLeft  ArrowRight
        if (e.key === 'ArrowLeft' || e.key === 'a') {
          // 往左一格
          if (!this.rowCollisionDetection('left')) {
            this.currentBlock.x -= this.unitSize;
          }
        } else if (e.key === 'ArrowRight' || e.key === 'd') {
          if (!this.rowCollisionDetection('right')) {
            this.currentBlock.x += this.unitSize;
          }
        } else if (e.key === 'j') {
          // 左旋
          let nextIndex = this.currentBlock.directionIndex - 1
          if (nextIndex < 0) {
            nextIndex = 3;
          }
          if (!this.rotateCollisionDetection(nextIndex))
            this.currentBlock.directionIndex = nextIndex

        } else if (e.key === 'k') {
          // 右旋
          let nextIndex = this.currentBlock.directionIndex + 1
          if (nextIndex > 3) {
            nextIndex = 0;
          }
          if (!this.rotateCollisionDetection(nextIndex))
            this.currentBlock.directionIndex = nextIndex
        } else if (e.key === 'r') {
          this.restartGame()
        } else if (e.key === 'i') {
          // 直接落下
          while (!this.collisionDetection()) {
            this.currentBlock.y += this.unitSize
          }
          // 完成后直接 出现下一块   保存然后继续
          if (this.contactStartTime) {
            this.updateCanvas();  // 落下方块
            this.line += this.removeLine()     // 消行
            this.lastImageData = this.screenCtx.getImageData(0, 0, this.width, this.height);
            this.randomList();
          }
        } else if (e.key === 'ArrowDown' || e.key === 's') {
          // 速降
          if (!this.collisionDetection()) {
            this.currentBlock.y += this.unitSize;
          }
        } else if (e.key === 'l') {
          // 与hold的方块交换
          let holdName = this.holdBlock.name;
          this.holdBlock.name = this.currentBlock.name;
          if (holdName === '') {
            this.randomList()
          } else {
            this.currentBlock = {
              name: holdName,
              x: this.unitSize * 4,
              y: 0,
              directionIndex: 0
            }
          }
          let name = this.holdBlock.name
          this.holdCtx.clearRect(0, 0, this.width, this.height)
          this['holdDraw' + name](0, 0);
        }
        this.updateCanvas()
        this.collisionDetection();
      })
    },
    updateCanvas() {
      // 这里要 保留上一个方块

      this.screenCtx.clearRect(0, 0, this.width, this.height)
      if (this.lastImageData) {
        this.screenCtx.putImageData(this.lastImageData, 0, 0)
      }
      this['draw' + this.currentBlock.name](this.currentBlock.x, this.currentBlock.y);
    },
    removeKeyboardListener() {
      document.removeEventListener('keydown', () => {
      })
    },
    // 检测游戏是否结束
    checkGameIsEnd() {
      // 游戏中 最上面的最中间那两块如果被填充，则判定为失败
      let imageData = this.screenCtx.getImageData(0, 0, this.width, this.unitSize);
      // * * * * ? ? * * * *
      if ((imageData.data[4 * this.unitSize * 4 + 3] === 255) &&
          imageData.data[5 * this.unitSize * 4 + 3] === 255) {
        this.gameIsEnd = true;
      }
    },
    restartGame() {
      this.gameIsEnd = false;
      this.screenCtx.clearRect(0, 0, this.width, this.height)
      this.frameTimeout = null;
      this.contactStartTime = false;
      this.lastImageData = false;
      this.randomList()
      this.run()
    },
    // 消行
    removeLine() {
      let imageData = this.screenCtx.getImageData(0, 0, this.width, this.height)
      let line = 0;
      for (let i = 0; i < 20; i++) {
        let lineFull = true;
        for (let j = 0; j < 10; j++) {
          if (imageData.data[(i * this.unitSize * (imageData.width * 4) + j * this.unitSize * 4) + 3] !== 255) {
            lineFull = false
          }
        }
        if (lineFull) {
          // 消除该行
          this.screenCtx.clearRect(0, i * this.unitSize, this.width, this.unitSize);
          // 在这里把行落下
          // 把这一行上面的方块都往下移
          let imageData = this.screenCtx.getImageData(0, 0, this.width, this.unitSize * i);
          this.screenCtx.putImageData(imageData, 0, this.unitSize)
          line++
        }
      }
      return line;
    }
  }
}
</script>
<style scoped lang="stylus">
bd = 1px solid #000000
.home
  width 900px
  height 800px
  border bd
  margin: 0 auto

  .top-area
    height 500px
    border-bottom bd
    display flex
    padding-top: 40px;
    position relative

    .score {
      position absolute
      left: 10px
      top: 10px
    }

    .screen
      width 520px
      height 460px
      border bd
      margin: 0 40px 0 40px

      .hold-block
        margin-top: 20px
        margin-left: 20px
        border bd

      .canvas
        float right
        border-left bd

    .top-right
      width 150px
      height 460px
      border bd
      currentWidth = 120px

      .current
        width currentWidth
        height @width
        border bd
        margin: 10px auto 20px

      .after
        width currentWidth
        height 2 * currentWidth + 30px
        border bd
        margin: 0 auto

  .bottom-area
    padding: 40px
    display flex

    .cross-button
      border bd
      width 150px
      height @width
      position relative
      triangleWidth = 50px

      .cross-button-item
        background #000000
        position absolute
        width triangleWidth
        height @width
        margin: auto;

        &::after
          content: ''
          width 0
          height 0
          position absolute

      .top
        top 0
        left 0
        right: 0;
        background #00ff29

        &::after
          top 100%
          left: 0
          border-top 0.5 * triangleWidth solid #00ff29
          border-right 0.5 * triangleWidth solid transparent
          border-left 0.5 * triangleWidth solid transparent

      .right
        top: 0
        bottom: 0
        right: 0
        background #f00

        &::after
          top 0
          left: -50%
          border-top 0.5 *triangleWidth solid transparent
          border-right 0.5 *triangleWidth solid #f00
          border-bottom 0.5 *triangleWidth solid transparent

      .bottom
        bottom: 0
        left: 0
        right: 0
        background #0065ff

        &::after
          top -50%
          left: 0
          border-bottom 0.5 *triangleWidth solid #0065ff
          border-right 0.5 *triangleWidth solid transparent
          border-left 0.5 *triangleWidth solid transparent

      .left
        left: 0
        top: 0
        bottom: 0
        background #ffdd00

        &::after
          top 0
          left: 100%
          border-top 0.5 *triangleWidth solid transparent
          border-bottom 0.5 *triangleWidth solid transparent
          border-left 0.5 *triangleWidth solid #ffdd00

    .function-button
      width: 200px
      height 150px
      border bd
      margin-left: 100px
      position relative

      .round-button
        border bd
        border-radius 50%
        width 50px
        height @width
        position absolute
        text-align center
        line-height @height
        font-size 20px
        font-weight 600

      .button-a
        top: 20px
        left: 60px

      .button-b
        top: 20px
        left: 130px

      .button-c
        top: 80px
        left: 30px

      .button-d
        top: 80px
        left: 100px
</style>
