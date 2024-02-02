<script setup lang="ts">
import { ref, onMounted, onUnmounted, reactive } from 'vue'

const canvasRef = ref<HTMLCanvasElement | null>(null)
const ctx = ref<CanvasRenderingContext2D | null>(null)

const config = reactive({
  hue: 0,
  bgFillColor: 'rgba(50, 50, 50, .05)',
  dirsCount: 6,
  stepToTurn: 12,
  dotSize: 4,
  dotsCount: 300,
  dotVelocity: 2,
  distance: 70,
  gradientLen: 5
})

const dirsList = Array.from({ length: config.dirsCount }, (_, i) => ({
  x: Math.cos((i * Math.PI) / config.dirsCount),
  y: Math.sin((i * Math.PI) / config.dirsCount)
}))

class Dot {
  pos = { x: 0, y: 0 }
  dir = ((Math.random() * 3) | 0) * 2
  step = 0

  redrawDot() {
    const xy = Math.abs(this.pos.x - config.dotSize / 2) + Math.abs(this.pos.y - config.dotSize / 2)
    const makeHue = (config.hue + xy / config.gradientLen) % 360
    const color = `hsl(${makeHue}, 100%, 50%)`
    const blur = config.dotSize - Math.sin(xy / 8) * 2
    const size = config.dotSize - Math.sin(xy / 9) * 2 + Math.sin(xy / 2)
    const x = this.pos.x - size / 2
    const y = this.pos.y - size / 2

    drawRect(color, x, y, size, size, color, blur, 'lighter')
  }

  moveDot() {
    this.step++
    this.pos.x += dirsList[this.dir].x * config.dotVelocity
    this.pos.y += dirsList[this.dir].y * config.dotVelocity
  }

  changeDir() {
    if (this.step % config.stepToTurn === 0) {
      this.dir =
        Math.random() > 0.5
          ? (this.dir + 1) % config.dirsCount
          : (this.dir + config.dirsCount - 1) % config.dirsCount
    }
  }

  killDot(id) {
    const percent = Math.random() * Math.exp(this.step / config.distance)
    if (percent > 100) {
      dotsList.splice(id, 1)
    }
  }
}

const dotsList = Array.from({ length: config.dotsCount }, () => new Dot())

function drawRect(
  color: string,
  x: number,
  y: number,
  w: number,
  h: number,
  shadowColor: string,
  shadowBlur: number,
  gco: string
) {
  ctx.value.globalCompositeOperation = gco
  ctx.value.shadowColor = shadowColor || 'black'
  ctx.value.shadowBlur = shadowBlur || 1
  ctx.value.fillStyle = color
  ctx.value?.fillRect(x, y, w, h)
}

function loop() {
  drawRect(
    config.bgFillColor,
    0,
    0,
    canvasRef.value?.width || 0,
    canvasRef.value?.height || 0,
    0,
    0,
    'normal'
  )
  dotsList.forEach((dot, id) => {
    dot.redrawDot()
    dot.moveDot()
    dot.changeDir()
    dot.killDot(id)
  })

  requestAnimationFrame(loop)
}

onMounted(() => {
  if (canvasRef.value) {
    ctx.value = canvasRef.value.getContext('2d')
    loop()
  }
})

onUnmounted(() => {
  // window.removeEventListener('resize', resizeCanvas)
})
</script>

<template>
  <main>
    <canvas ref="canvasRef"></canvas>
    <SideBar />
  </main>
</template>

<style>
* {
  font-family: Arial, sans-serif;
  box-sizing: border-box;
  margin: 0;
}
main {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  /*background-color: var(--main-bg-color-secondary);*/
  overflow: hidden;
}
canvas {
  background: rgb(30, 30, 30);
  height: 100%;
  width: 100%;
}
</style>
