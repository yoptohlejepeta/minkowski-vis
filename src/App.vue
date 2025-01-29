<script setup>
import { RouterLink, RouterView } from 'vue-router'
import { Button } from 'primevue';
import Canvas from './components/Canvas.vue';
import 'primeicons/primeicons.css';
import Dock from 'primevue/dock';


var items = [
  { label: 'Add polygon', icon: 'https://www.primefaces.org/primevue/showcase/assets/showcase/images/dock/plus.svg' },
  { label: 'Remove polygon', icon: 'https://www.primefaces.org/primevue/showcase/assets/showcase/images/dock/minus.svg' },
  { label: 'Clear all', icon: 'https://www.primefaces.org/primevue/showcase/assets/showcase/images/dock/close.svg' }
];

var position = 'bottom';
</script>

<template>
  <header style="display: flex; align-items: center; justify-content: space-between;">
    <Button icon="pi pi-home" aria-label="Save" />
    <h1>
      Minkowski Sum
    </h1>
    <Button icon="pi pi-save" aria-label="Save" />
  </header>

  <Canvas/>

  <Dock :model="items" :position="position">
    <template #itemicon="{ item }">
        <Button icon="pi pi-plus" aria-label="Save" />
    </template>
</Dock>

  <!-- <Button label="Add polygon"></Button> -->
</template>

<style scoped>

h1 {
  color: black;
}

header {
  /* background-color: rgba(255, 255, 255, 0.5); */
  backdrop-filter: blur(10px);
  color: white;
  padding: 10px;
  text-align: center;
  border-radius: 15px;
}

</style>
