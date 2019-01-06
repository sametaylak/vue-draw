<template>
  <div class="canvas-wrapper" ref="canvasWrapper">
    <div class="draw-area">
      <canvas id="canvas" ref="canvas" :width="width" :height="height"></canvas>
      <canvas id="cursor" ref="cursor" :width="width" :height="height"></canvas>
    </div>
    <ul class="tools">
      <li id="tool-pencil" :class="{ active: selectedToolIdx === 0 }" @click="changeTool(0)">
        <img src="@/assets/edit.svg"/>
      </li>
      <li id="tool-eraser" :class="{ active: selectedToolIdx === 1 }" @click="changeTool(1)">
        <img src="@/assets/eraser.svg"/>
      </li>
      <li id="tool-color-palette" @click="showColorPalette()">
        <img src="@/assets/color-palette.svg"/>
      </li>
      <li id="tool-download" @click="download()">
        <img src="@/assets/download.svg"/>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'CanvasDraw',
  props: {
    brushSize: {
      type: Number,
      default: 12,
    },
    width: {
      type: Number,
      default: 640,
    },
    height: {
      type: Number,
      default: 480,
    },
    outputName: {
      type: String,
      default: 'canvas',
    },
  },
  data() {
    return {
      canvasContext: null,
      cursorContext: null,
      isDrawing: false,
      lastX: 0,
      lastY: 0,
      tools: [
        {
          name: 'Pencil',
          color: '#000000',
        },
        {
          name: 'Eraser',
        },
      ],
      selectedToolIdx: 0,
    };
  },
  mounted() {
    this.setCanvas();
    this.bindEvents();
  },
  methods: {
    setCanvas() {
      this.$refs.canvasWrapper.style.gridTemplateColumns = `${this.width}px 30px`;
      this.$refs.canvasWrapper.style.width = `${this.width + 30}px`;
      this.$refs.canvasWrapper.style.height = `${this.height}px`;

      this.canvasContext = this.$refs.canvas.getContext('2d');
      this.canvasContext.lineJoin = 'round';
      this.canvasContext.lineCap = 'round';
      this.canvasContext.lineWidth = this.brushSize;
      this.canvasContext.strokeStyle = this.tools[this.selectedToolIdx].color;

      this.cursorContext = this.$refs.cursor.getContext('2d');
    },
    bindEvents() {
      this.$refs.canvas.addEventListener('mousedown', (event) => {
        this.isDrawing = true;
        [this.lastX, this.lastY] = [event.offsetX, event.offsetY];
      });
      this.$refs.canvas.addEventListener('mousemove', this.draw);
      this.$refs.canvas.addEventListener('mouseup', () => this.isDrawing = false);
      this.$refs.canvas.addEventListener('mouseout', () => this.isDrawing = false);
    },
    changeTool(tool) {
      this.selectedToolIdx = tool;
    },
    draw(event) {
      this.drawCursor(event);
      if (!this.isDrawing) return;

      if (this.tools[this.selectedToolIdx].name === 'Eraser') {
        this.canvasContext.globalCompositeOperation = 'destination-out';
      } else {
        this.canvasContext.globalCompositeOperation = 'source-over';
        this.canvasContext.strokeStyle = this.tools[this.selectedToolIdx].color;
      }

      this.canvasContext.beginPath();
      this.canvasContext.moveTo(this.lastX, this.lastY);
      this.canvasContext.lineTo(event.offsetX, event.offsetY);
      this.canvasContext.stroke();
      [this.lastX, this.lastY] = [event.offsetX, event.offsetY];
    },
    drawCursor(event) {
      this.cursorContext.beginPath();
      this.cursorContext.ellipse(
        event.offsetX, event.offsetY,
        this.brushSize, this.brushSize,
        Math.PI / 4, 0, 2 * Math.PI
      );
      this.cursorContext.stroke();
      setTimeout(() => {
        this.cursorContext.clearRect(0, 0, this.width, this.height);
      }, 100);
    },
    showColorPalette() {
      const colorPalette = document.createElement('input');
      colorPalette.addEventListener('change', (event) => {
        this.tools[0].color = event.target.value;
      });
      colorPalette.type = 'color';
      colorPalette.value = this.tools[0].color;
      colorPalette.click();
    },
    download() {
      const link = document.createElement('a');
      link.download = `${this.outputName}.png`;
      link.href = this.$refs.canvas.toDataURL()
      link.click();
    },
  }
}
</script>
<style scoped>
.canvas-wrapper {
  display: grid;
  position: relative;
}
#canvas {
  background-color: #f9f9f9;
  z-index: 0;
}
#cursor {
  z-index: 1;
  pointer-events: none;
}
.active {
  background-color: #dea878 !important;
}
.tools {
  margin: 0;
  padding: 0;
}
.tools li{
  padding: 4px;
  background-color: #c8c8c8;
  border-left: 1px solid #abaaaa;
}
.tools li:not(:last-child) {
  border-bottom: 1px solid #abaaaa;
}
.draw-area canvas {
  position: absolute;
  left: 0;
  top: 0;
  border: 2px solid #c8c8c8;
  border-radius: 10px 0 10px 10px;
}
</style>
