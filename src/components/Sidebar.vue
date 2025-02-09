<template>
    <div class="w-64 bg-gray-800 text-white p-4">
      <h2 class="text-xl font-bold mb-4">Polygons</h2>
      <ul>
        <li 
          v-for="(polygon, index) in polygons" 
          :key="polygon.id"
          class="mb-2 p-2 rounded cursor-pointer"
          :class="{ 'bg-gray-700': index === activePolygonIndex }"
          @click="selectPolygon(index)"
        >
          <div class="flex items-center">
            <div 
              class="w-4 h-4 rounded-full mr-2" 
              :style="{ backgroundColor: polygon.color }"
            ></div>
            <span>Polygon {{ polygon.id }}</span>
          </div>
          <div class="text-xs mt-1">
            Points: {{ polygon.points.length }}
          </div>
        </li>
      </ul>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue';
  
  const props = defineProps(['polygons']);
  const emit = defineEmits(['select-polygon']);
  
  const activePolygonIndex = ref(0);
  
  const selectPolygon = (index) => {
    console.log('selectPolygon', index);
    activePolygonIndex.value = index;
    emit('select-polygon', index);
  };
  </script>