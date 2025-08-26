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
            <button class="vue--draw__tools__item" @click="selectPencil" :class="{active: currentTool === eToolType.PENCIL}">✏️
            </button>
            <button class="vue--draw__tools__item" @click="selectPen">🖊️</button>
            <button class="vue--draw__tools__item" @click="selectMarker">🖍️</button>
            <button class="vue--draw__tools__item" @click="selectEraser">🩹</button>
            <button class="vue--draw__tools__item" @click="clearCanvas">X</button>
            <input type="color" v-model="pencilColor">
            <label>Size: {{ toolSize }}</label>
            <input type="range" min="1" max="50" v-model="toolSize">
        </div>
    </div>
</template>

<script setup lang="ts">
import {ref, onMounted, watch} from 'vue'
import {eToolType} from "./enums/tool-enums.ts";

const canvasRef = ref<HTMLCanvasElement | null>(null)
const currentTool = ref<eToolType | null>(null)

//TODO эти let переменные сделать тоже ref<>, а лучше всего вообще через reactive
let ctx: CanvasRenderingContext2D | null = null
let drawing = false
let lastX = 0
let lastY = 0

//TODO сделай что бы сюда применялся изначально --vue-draw-black из :root
const pencilColor = ref('#000000')
const toolSize = ref(2)

const selectEraser = () => {
    currentTool.value = eToolType.ERASER
    if (ctx) {
        ctx.globalCompositeOperation = 'destination-out'
        ctx.lineWidth = 20
        toolSize.value = 5
        console.log("eraser work")
    }
}

const selectPencil = () => {
    currentTool.value = eToolType.PENCIL
    if (ctx) {
        ctx.globalCompositeOperation = 'source-over'
        ctx.strokeStyle = pencilColor.value
        ctx.lineWidth = 2
        toolSize.value = 2
        console.log("pencil work")
    }
}

const selectPen = () => {
    currentTool.value = eToolType.PEN
    if (ctx) {
        ctx.globalCompositeOperation = 'source-over'
        ctx.strokeStyle = pencilColor.value
        ctx.lineWidth = 1
        toolSize.value = 1
        ctx.lineCap = 'round'
    }
}

const selectMarker = () => {
    currentTool.value = eToolType.MARKER
    if (ctx) {
        ctx.globalCompositeOperation = 'source-over'
        ctx.strokeStyle = pencilColor.value
        ctx.lineWidth = 15
        toolSize.value = 15
        ctx.lineCap = 'round'
        ctx.lineJoin = 'round'
        ctx.strokeStyle = hexToRgba(pencilColor.value, 0.3)
    }
}

function hexToRgba(hex: string, alpha: number) {
    const r = parseInt(hex.slice(1, 3), 16)
    const g = parseInt(hex.slice(3, 5), 16)
    const b = parseInt(hex.slice(5, 7), 16)
    return `rgba(${r},${g},${b},${alpha})`
}

const getMousePos = (e: MouseEvent) => {
    if (!canvasRef.value) return {x: 0, y: 0}
    const rect = canvasRef.value.getBoundingClientRect()
    return {
        x: (e.clientX - rect.left) * (canvasRef.value.width / rect.width),
        y: (e.clientY - rect.top) * (canvasRef.value.height / rect.height)
    }
}

const onPointerDown = (e: MouseEvent) => {
    if (!ctx || !currentTool) return
    drawing = true
    //ctx.strokeStyle=pencilColor.value
    const pos = getMousePos(e)
    lastX = pos.x
    lastY = pos.y
    ctx.beginPath()
    ctx.moveTo(lastX, lastY)
}

const onPointerUp = () => {
    drawing = false
}

const onPointerMove = (e: MouseEvent) => {
    if (!drawing || !ctx || !currentTool) return
    const pos = getMousePos(e)
    const midX = (lastX + pos.x) / 2
    const midY = (lastY + pos.y) / 2
    ctx.quadraticCurveTo(lastX, lastY, midX, midY)
    ctx.stroke()
    lastX = pos.x
    lastY = pos.y
}

const clearCanvas = () => {
    if (ctx && canvasRef.value) {
        ctx.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height)
    }
}

watch(pencilColor, (newColor) => {
    if (!ctx) return
    if (currentTool.value === eToolType.PENCIL || currentTool.value === eToolType.PEN) {
        ctx.strokeStyle = newColor
    } else if (currentTool.value === eToolType.MARKER) {
        ctx.strokeStyle = hexToRgba(newColor, 0.3)
    }
})

watch(toolSize,(newSize)=>{
    if(!ctx) return
    ctx.lineWidth=newSize
})

onMounted(() => {
    if (canvasRef.value) {
        canvasRef.value.width = window.innerWidth
        canvasRef.value.height = window.innerHeight
        ctx = canvasRef.value.getContext('2d')
        if (ctx) {
            ctx.lineWidth = 2
            ctx.lineCap = 'round'
            ctx.strokeStyle = pencilColor.value
        }
    }

    //TODO добавить в onBeforeUnmount удаление этого event listener
    window.addEventListener("resize", () => {
        if (canvasRef.value) {
            canvasRef.value.width = window.innerWidth
            canvasRef.value.height = window.innerHeight
        }
    })
})
</script>