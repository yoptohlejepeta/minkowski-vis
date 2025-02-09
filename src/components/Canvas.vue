<template>
  <div class="w-full h-full relative">
    <canvas
      ref="canvasRef"
      @click="handleCanvasClick"
      @mousemove="handleMouseMove"
      @mousedown="handleMouseDown"
      @mouseup="handleMouseUp"
      class="w-full h-full"
    ></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';

const props = defineProps({
  polygons: {
    type: Array,
    required: true
  },
  activePolygonIndex: {
    type: Number,
    required: true
  }
});

const emit = defineEmits(['add-point', 'update-point', 'delete-point', 'move-polygon']);

const canvasRef = ref(null);
const ctx = ref(null);
const isDragging = ref(false);
const draggedItem = ref(null);

onMounted(() => {
  const canvas = canvasRef.value;
  canvas.width = canvas.offsetWidth;
  canvas.height = canvas.offsetHeight;
  ctx.value = canvas.getContext('2d');
  drawPolygons();
});

watch([() => props.polygons, () => props.activePolygonIndex], drawPolygons, { deep: true });

function drawPolygons() {
  if (!ctx.value) return;

  ctx.value.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height);

  // Draw axes
  ctx.value.strokeStyle = '#ccc';
  ctx.value.beginPath();
  ctx.value.moveTo(0, canvasRef.value.height / 2);
  ctx.value.lineTo(canvasRef.value.width, canvasRef.value.height / 2);
  ctx.value.moveTo(canvasRef.value.width / 2, 0);
  ctx.value.lineTo(canvasRef.value.width / 2, canvasRef.value.height);
  ctx.value.stroke();

  props.polygons.forEach((polygon, index) => {
    ctx.value.strokeStyle = polygon.color;
    ctx.value.fillStyle = `${polygon.color}40`;
    ctx.value.lineWidth = index === props.activePolygonIndex ? 3 : 1;

    if (polygon.points.length > 0) {
      ctx.value.beginPath();
      polygon.points.forEach((point, i) => {
        if (i === 0) {
          ctx.value.moveTo(point.x, point.y);
        } else {
          ctx.value.lineTo(point.x, point.y);
        }
      });
      if (polygon.points.length > 2) {
        ctx.value.closePath();
      }
      ctx.value.stroke();
      ctx.value.fill();

      // Draw points
      polygon.points.forEach((point) => {
        ctx.value.fillStyle = polygon.color;
        ctx.value.beginPath();
        ctx.value.arc(point.x, point.y, 4, 0, Math.PI * 2);
        ctx.value.fill();
      });

      // Draw center point for moving the polygon
      const center = getPolygonCenter(polygon.points);
      ctx.value.fillStyle = 'white';
      ctx.value.strokeStyle = polygon.color;
      ctx.value.lineWidth = 2;
      ctx.value.beginPath();
      ctx.value.arc(center.x, center.y, 8, 0, Math.PI * 2);
      ctx.value.fill();
      ctx.value.stroke();
    }
  });
}

function getPolygonCenter(points) {
  const x = points.reduce((sum, p) => sum + p.x, 0) / points.length;
  const y = points.reduce((sum, p) => sum + p.y, 0) / points.length;
  return { x, y };
}

function handleCanvasClick(event) {
  if (!isDragging.value) {
    const rect = canvasRef.value.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;
    emit('add-point', x, y);
  }
}

function handleMouseMove(event) {
  if (isDragging.value && draggedItem.value) {
    const rect = canvasRef.value.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;
    
    if (draggedItem.value.type === 'point') {
      emit('update-point', draggedItem.value.polygonIndex, draggedItem.value.pointIndex, x, y);
    } else if (draggedItem.value.type === 'polygon') {
      const dx = x - draggedItem.value.lastX;
      const dy = y - draggedItem.value.lastY;
      emit('move-polygon', draggedItem.value.polygonIndex, dx, dy);
      draggedItem.value.lastX = x;
      draggedItem.value.lastY = y;
    }
  }
}

function handleMouseDown(event) {
  const rect = canvasRef.value.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;
  
  for (let i = 0; i < props.polygons.length; i++) {
    const polygon = props.polygons[i];
    
    // Check if clicking on the center point to move the polygon
    const center = getPolygonCenter(polygon.points);
    if (Math.abs(center.x - x) < 10 && Math.abs(center.y - y) < 10) {
      isDragging.value = true;
      draggedItem.value = { type: 'polygon', polygonIndex: i, lastX: x, lastY: y };
      return;
    }
    
    // Check if clicking on a point to move it
    for (let j = 0; j < polygon.points.length; j++) {
      const point = polygon.points[j];
      if (Math.abs(point.x - x) < 5 && Math.abs(point.y - y) < 5) {
        isDragging.value = true;
        draggedItem.value = { type: 'point', polygonIndex: i, pointIndex: j };
        return;
      }
    }
  }
}

function handleMouseUp() {
  isDragging.value = false;
  draggedItem.value = null;
}
</script>