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
import {ref, reactive, onMounted,onBeforeUnmount, watch} from 'vue'
import {eToolType} from "./enums/tool-enums.ts";

const canvasRef = ref<HTMLCanvasElement | null>(null)
const currentTool = ref<eToolType | null>(null)

//все через state☑️☑️☑️
const state = reactive({
    ctx: null as CanvasRenderingContext2D | null,
    drawing: false,
    lastX: 0,
    lastY: 0
})

//TODO сделай что бы сюда применялся изначально --vue-draw-black из :root :☑️☑️☑️☑️
const pencilColor = ref(
    getComputedStyle(document.documentElement).getPropertyValue("--vue-draw-black").trim()
)
const toolSize = ref(2)

const selectEraser = () => {
    currentTool.value = eToolType.ERASER
    if (state.ctx) {
        state.ctx.globalCompositeOperation = 'destination-out'
        state.ctx.lineWidth = 20
        toolSize.value = 5
        console.log("eraser work")
    }
}

const selectPencil = () => {
    currentTool.value = eToolType.PENCIL
    if (state.ctx) {
        state.ctx.globalCompositeOperation = 'source-over'
        state.ctx.strokeStyle = pencilColor.value
        state.ctx.lineWidth = 2
        toolSize.value = 2
        console.log("pencil work")
    }
}

const selectPen = () => {
    currentTool.value = eToolType.PEN
    if (state.ctx) {
        state.ctx.globalCompositeOperation = 'source-over'
        state.ctx.strokeStyle = pencilColor.value
        state.ctx.lineWidth = 1
        toolSize.value = 1
        state.ctx.lineCap = 'round'
    }
}

const selectMarker = () => {
    currentTool.value = eToolType.MARKER
    if (state.ctx) {
        state.ctx.globalCompositeOperation = 'source-over'
        state.ctx.lineWidth = 15
        toolSize.value = 15
        state.ctx.lineCap = 'round'
        state.ctx.lineJoin = 'round'
        state.ctx.strokeStyle = hexToRgba(pencilColor.value, 0.3)
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
    if (!state.ctx || !currentTool.value) return
    state.drawing = true
    const pos = getMousePos(e)
    state.lastX = pos.x
    state.lastY = pos.y
    state.ctx.beginPath()
    state.ctx.moveTo(state.lastX, state.lastY)
}

const onPointerUp = () => {
    state.drawing = false
}

const onPointerMove = (e: MouseEvent) => {
    if (!state.drawing || !state.ctx || !currentTool.value) return
    const pos = getMousePos(e)
    const midX = (state.lastX + pos.x) / 2
    const midY = (state.lastY + pos.y) / 2
    state.ctx.quadraticCurveTo(state.lastX, state.lastY, midX, midY)
    state.ctx.stroke()
    state.lastX = pos.x
    state.lastY = pos.y
}

const clearCanvas = () => {
    if (state.ctx && canvasRef.value) {
        state.ctx.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height)
    }
}

watch(pencilColor, (newColor) => {
    if (!state.ctx) return
    if (currentTool.value === eToolType.PENCIL || currentTool.value === eToolType.PEN) {
        state.ctx.strokeStyle = newColor
    } else if (currentTool.value === eToolType.MARKER) {
        state.ctx.strokeStyle = hexToRgba(newColor, 0.3)
    }
})

watch(toolSize,(newSize)=>{
    if(!state.ctx) return
    state.ctx.lineWidth=newSize
})
//TODO добавить в onBeforeUnmount удаление этого event listener ☑️☑️☑️☑️☑️☑️
const handleResize = () => {
    if(!canvasRef.value) return
    canvasRef.value.width = window.innerWidth
    canvasRef.value.height = window.innerHeight
}
onMounted(() => {
    if (canvasRef.value) {
        canvasRef.value.width = window.innerWidth
        canvasRef.value.height = window.innerHeight
        state.ctx = canvasRef.value.getContext('2d')
        if (state.ctx) {
            state.ctx.lineWidth = 2
            state.ctx.lineCap = 'round'
            state.ctx.strokeStyle = pencilColor.value
        }
        window.addEventListener('resize', handleResize)
    }
})
onBeforeUnmount(() => {
    window.addEventListener('resize', handleResize)
})

</script>
