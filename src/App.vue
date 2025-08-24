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
      <button class="vue--draw__tools__item" @click="">🖊️</button>
      <button class="vue--draw__tools__item"    >🖍️</button>
      <button class="vue--draw__tools__item" @click="selectEraser">🩹</button>
      <button class="vue--draw__tools__item" @click="clearCanvas">X</button>
      <input type="color" v-model="pencilColor">
      <label>Size: {{toolSize}}</label>
      <input type="range" min="1" max="50" v-model="toolSize">
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'

const canvasRef = ref<HTMLCanvasElement | null>(null)
let ctx: CanvasRenderingContext2D | null = null
let drawing = false
let currentTool: 'pencil'|'eraser' | null = null
let lastX = 0
let lastY = 0
const pencilColor = ref('#000000')
const toolSize=ref(2)
const selectEraser=()=>{
    currentTool='eraser'
    if(ctx){
        ctx.globalCompositeOperation='destination-out'
        ctx.lineWidth=20
        console.log("eraser work")
    }
}
const selectPencil = () => {
    currentTool = 'pencil'
    if(ctx){
        ctx.globalCompositeOperation='source-over'
        ctx.strokeStyle=pencilColor.value
        ctx.lineWidth=2
        console.log("pencil work")
    }
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
    if (!ctx || !currentTool) return
    drawing = true
    ctx.strokeStyle=pencilColor.value
    const pos = getMousePos(e)
    lastX = pos.x
    lastY = pos.y
}
const onPointerUp = () => {
    drawing = false
}
const onPointerMove = (e: MouseEvent) => {
    if (!drawing || !ctx || !currentTool) return
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
window.addEventListener("resize", ()=>{
    if(canvasRef.value) {
        canvasRef.value.width = window.innerWidth
        canvasRef.value.height = window.innerHeight
    }})
onMounted(() => {
    if (canvasRef.value) {
        canvasRef.value.width=window.innerWidth
        canvasRef.value.height=window.innerHeight
        ctx = canvasRef.value.getContext('2d')
    if (ctx) {
      ctx.lineWidth = 2
      ctx.lineCap = 'round'
      ctx.strokeStyle = pencilColor.value
    }
  }
})
watch(pencilColor,(newColor)=>{
    if(ctx&&currentTool==='pencil'){
        ctx.strokeStyle=newColor;
    }
})
watch(toolSize,(newSize)=>{
    if(!ctx) return
    ctx.lineWidth=newSize
})
</script>
