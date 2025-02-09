<template>
    <div class="bg-gray-200 p-4 flex justify-between items-center">
      <button 
        @click="$emit('new-polygon')"
        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
      >
        New Polygon
      </button>
      <button 
        @click="$emit('calculate-minkowski-sum')"
        class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded"
      >
        Calculate Minkowski Sum
      </button>
    </div>
  </template>
  
  <script setup>
  // No additional logic needed for this component
  </script>