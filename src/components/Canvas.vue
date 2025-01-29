<template>
    <canvas
      ref="canvas"
      @click="show_point([$event.offsetX, $event.offsetY])"
    ></canvas>
  </template>
  
  <script setup>
  import { onMounted, ref } from 'vue';
  
  const canvas = ref(null);
  
  function show_point(point) {
    console.log(window.innerWidth / 2 - point[0], window.innerHeight / 2 - point[1]);
  }
  
  function drawAxes() {
    const ctx = canvas.value.getContext("2d");
    const width = canvas.value.width = window.innerWidth;
    const height = canvas.value.height = window.innerHeight;
    const centerX = width / 2;
    const centerY = height / 2;
  
    ctx.clearRect(0, 0, width, height);
  
    // Draw x and y axis
    ctx.strokeStyle = "black";
    ctx.lineWidth = 2;
    
    // Y-axis
    ctx.beginPath();
    ctx.moveTo(centerX, 0);
    ctx.lineTo(centerX, height);
    ctx.stroke();
  
    // X-axis
    ctx.beginPath();
    ctx.moveTo(0, centerY);
    ctx.lineTo(width, centerY);
    ctx.stroke();
  }
  
  onMounted(() => {
    drawAxes();
    window.addEventListener('resize', drawAxes);
  });
  </script>
  
  <style scoped>
  canvas {
    position: absolute;
    left: 0;
    top: 0;
    width: 100vw;
    height: 100vh;
    z-index: -1;
  }
  </style>
  