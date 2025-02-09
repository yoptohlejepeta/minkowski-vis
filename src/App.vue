<template>
  <div class="min-h-screen bg-background flex">
    <!-- Sidebar -->
    <aside class="w-64 bg-card text-card-foreground p-4">
      <h2 class="text-lg font-semibold mb-4">Polygons</h2>
      <ul class="space-y-2">
        <li v-for="(polygon, index) in polygons" :key="polygon.id" 
            class="p-2 rounded cursor-pointer hover:bg-accent hover:text-accent-foreground"
            :class="{ 'bg-accent text-accent-foreground': index === activePolygonIndex }">
          <div class="flex items-center justify-between">
            <div class="flex items-center" @click="selectPolygon(index)">
              <div class="w-4 h-4 rounded-full mr-2" :style="{ backgroundColor: polygon.color }"></div>
              <span>Polygon {{ polygon.id }}</span>
            </div>
            <Button @click="removePolygon(index)" variant="destructive" size="sm">
              Remove
            </Button>
          </div>
          <div class="text-xs mt-1">
            Points: {{ polygon.points.length }}
          </div>
        </li>
      </ul>
    </aside>

    <!-- Main content -->
    <main class="flex-1 flex flex-col">
      <!-- Canvas -->
      <div class="flex-1 bg-muted">
        <Canvas 
          :polygons="polygons" 
          :activePolygonIndex="activePolygonIndex"
          @add-point="addPoint"
          @update-point="updatePoint"
          @delete-point="deletePoint"
          @move-polygon="movePolygon"
        />
      </div>

      <!-- Control panel -->
      <div class="bg-card text-card-foreground p-4 flex justify-between">
        <Button @click="newPolygon">New Polygon</Button>
        <Button @click="calculateMinkowskiSum" variant="secondary">Calculate Minkowski Sum</Button>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { Button } from '@/components/ui/button';
import Canvas from '@/components/Canvas.vue';

const polygons = ref([
  { id: 1, points: [], color: '#FF5733' },
  { id: 2, points: [], color: '#33FF57' }
]);

const activePolygonIndex = ref(0);

const selectPolygon = (index) => {
  activePolygonIndex.value = index;
};

const addPoint = (x, y) => {
  polygons.value[activePolygonIndex.value].points.push({ x, y });
};

const updatePoint = (polygonIndex, pointIndex, x, y) => {
  polygons.value[polygonIndex].points[pointIndex] = { x, y };
};

const deletePoint = (polygonIndex, pointIndex) => {
  polygons.value[polygonIndex].points.splice(pointIndex, 1);
};

const movePolygon = (polygonIndex, dx, dy) => {
  polygons.value[polygonIndex].points = polygons.value[polygonIndex].points.map(point => ({
    x: point.x + dx,
    y: point.y + dy
  }));
};

const newPolygon = () => {
  const newId = Math.max(...polygons.value.map(p => p.id), 0) + 1;
  polygons.value.push({ id: newId, points: [], color: getRandomColor() });
  activePolygonIndex.value = polygons.value.length - 1;
};

const removePolygon = (index) => {
  polygons.value.splice(index, 1);
  if (activePolygonIndex.value >= polygons.value.length) {
    activePolygonIndex.value = Math.max(0, polygons.value.length - 1);
  }
};

const calculateMinkowskiSum = () => {
  if (polygons.value.length < 2) return;
  
  const result = minkowskiSum(polygons.value[0].points, polygons.value[1].points);
  polygons.value.push({ 
    id: Math.max(...polygons.value.map(p => p.id), 0) + 1,
    points: result, 
    color: '#5733FF',
    isMinkowskiSum: true
  });
  activePolygonIndex.value = polygons.value.length - 1;
};

const getRandomColor = () => {
  return '#' + Math.floor(Math.random()*16777215).toString(16);
};

const minkowskiSum = (poly1, poly2) => {
  const result = [];
  for (const p1 of poly1) {
    for (const p2 of poly2) {
      result.push({ x: p1.x + p2.x, y: p1.y + p2.y });
    }
  }
  return convexHull(result);
};

const convexHull = (points) => {
  // Implement Graham scan algorithm for convex hull
  // This is a simplified version and may not work for all cases
  points.sort((a, b) => a.x - b.x || a.y - b.y);
  const lower = [];
  for (let i = 0; i < points.length; i++) {
    while (lower.length >= 2 && cross(lower[lower.length - 2], lower[lower.length - 1], points[i]) <= 0) {
      lower.pop();
    }
    lower.push(points[i]);
  }
  const upper = [];
  for (let i = points.length - 1; i >= 0; i--) {
    while (upper.length >= 2 && cross(upper[upper.length - 2], upper[upper.length - 1], points[i]) <= 0) {
      upper.pop();
    }
    upper.push(points[i]);
  }
  upper.pop();
  lower.pop();
  return lower.concat(upper);
};

const cross = (o, a, b) => {
  return (a.x - o.x) * (b.y - o.y) - (a.y - o.y) * (b.x - o.x);
};
</script>