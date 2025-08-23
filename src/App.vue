<template>
    <div class="vue--draw__container">
      <canvas class="vue--draw__canvas" ref="vue--draw__canvas"></canvas>
        <div class="vue--draw__tools">
            <button class="vue--draw__tools__item">
                ✏️
            </button>
            <button class="vue--draw__tools__item">
                🖊️
            </button>
            <button class="vue--draw__tools__item">
                🖍️
            </button>
            <button class="vue--draw__tools__item" @click="ClearCanvas()">
                🩹
            </button>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
const canvasRef=ref<HTMLCanvasElement|null>(null);
const ClearCanvas = () => {
  if (canvasRef.value) {
    const ctx=canvasRef.value.getContext('2d');
    if(ctx){
      ctx.clearRect(0,0, canvasRef.value.width, canvasRef.value.height);
    }
  }
}
</script>