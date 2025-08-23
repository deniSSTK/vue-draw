<template>
  <div class="vue--draw__container">
    <div class="vue--draw__cont"
         @mousedown="onPointerDown"
         @mouseup="onPointerUp"
         @mouseleave="onPointerUp"
         @mousemove="onPointerMove">
      <canvas class="vue--draw__canvas" ref="canvasRef"></canvas>
    </div>
    <div class="vue--draw__tools">
      <button class="vue--draw__tools__item" @click="selectPencil">
        ✏️
      </button>
      <button class="vue--draw__tools__item">🖊️</button>
      <button class="vue--draw__tools__item">🖍️</button>
      <button class="vue--draw__tools__item" @click="clearCanvas">
        🩹
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

const canvasRef = ref<HTMLCanvasElement | null>(null)
let ctx: CanvasRenderingContext2D | null = null
let drawing = false
let currentTool: 'pencil' | null = null
let lastX = 0
let lastY = 0

const selectPencil = () => {
  currentTool = 'pencil'
}

const getMousePos = (e: MouseEvent) => {
  if (!canvasRef.value) return { x: 0, y: 0 }
  const rect = canvasRef.value.getBoundingClientRect()
  return {
    x: (e.clientX - rect.left) * (canvasRef.value.width / rect.width),
    y: (e.clientY - rect.top) * (canvasRef.value.height / rect.height)
  }
}

const onPointerDown = (e: MouseEvent) => {
  if (currentTool !== 'pencil' || !ctx) return
  drawing = true
  const pos = getMousePos(e)
  lastX = pos.x
  lastY = pos.y
}

const onPointerUp = () => {
  drawing = false
}

const onPointerMove = (e: MouseEvent) => {
  if (!drawing || currentTool !== 'pencil' || !ctx) return
  const pos = getMousePos(e)
  ctx.beginPath()
  ctx.moveTo(lastX, lastY)
  ctx.lineTo(pos.x, pos.y)
  ctx.stroke()
  lastX = pos.x
  lastY = pos.y
}

const clearCanvas = () => {
  if (ctx && canvasRef.value) {
    ctx.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height)
  }
}

onMounted(() => {
  if (canvasRef.value) {
    canvasRef.value.width = 1920
    canvasRef.value.height = 1280
    ctx = canvasRef.value.getContext('2d')
    if (ctx) {
      ctx.lineWidth = 2
      ctx.lineCap = 'round'
      ctx.strokeStyle = 'black'
    }
  }
})
</script>
