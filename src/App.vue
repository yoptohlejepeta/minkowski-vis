<script setup>
import { RouterLink, RouterView } from 'vue-router'
import Button from "primevue/button";

function show_point(point) {
  console.log(point);
}

</script>

<template>
  <h1>
    Minkowski Sum
  </h1>

  <canvas
  @click="show_point([$event.offsetX, $event.offsetY])"
  style="border:1px solid #000000;"
  ></canvas>
  <!-- <RouterView /> -->
</template>

<style scoped>
</style>
