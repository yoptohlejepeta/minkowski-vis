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

  const canvas = canvasRef.value;
  ctx.value.clearRect(0, 0, canvas.width, canvas.height);

  // Draw axes
  ctx.value.strokeStyle = '#ccc';
  ctx.value.beginPath();
  ctx.value.moveTo(0, canvas.height / 2);
  ctx.value.lineTo(canvas.width, canvas.height / 2);
  ctx.value.moveTo(canvas.width / 2, 0);
  ctx.value.lineTo(canvas.width / 2, canvas.height);
  ctx.value.stroke();

  // Add text for axes
  ctx.value.fillStyle = '#666';
  ctx.value.font = '12px Arial';
  ctx.value.fillText('Y', canvas.width / 2 + 5, 15);
  ctx.value.fillText('X', canvas.width - 15, canvas.height / 2 - 5);

  props.polygons.forEach((polygon, index) => {
    ctx.value.strokeStyle = polygon.color;
    ctx.value.fillStyle = `${polygon.color}40`;
    ctx.value.lineWidth = index === props.activePolygonIndex ? 3 : 1;

    if (polygon.points.length > 0) {
      ctx.value.beginPath();
      polygon.points.forEach((point, i) => {
        const canvasX = canvas.width / 2 + point.x;
        const canvasY = canvas.height / 2 - point.y;
        if (i === 0) {
          ctx.value.moveTo(canvasX, canvasY);
        } else {
          ctx.value.lineTo(canvasX, canvasY);
        }
      });
      if (polygon.points.length > 2) {
        ctx.value.closePath();
      }
      ctx.value.stroke();
      ctx.value.fill();

      // Draw points
      polygon.points.forEach((point) => {
        const canvasX = canvas.width / 2 + point.x;
        const canvasY = canvas.height / 2 - point.y;
        ctx.value.fillStyle = polygon.color;
        ctx.value.beginPath();
        ctx.value.arc(canvasX, canvasY, 4, 0, Math.PI * 2);
        ctx.value.fill();
      });

      // Draw center point for moving the polygon
      const center = getPolygonCenter(polygon.points);
      const canvasCenterX = canvas.width / 2 + center.x;
      const canvasCenterY = canvas.height / 2 - center.y;
      ctx.value.fillStyle = 'white';
      ctx.value.strokeStyle = polygon.color;
      ctx.value.lineWidth = 2;
      ctx.value.beginPath();
      ctx.value.arc(canvasCenterX, canvasCenterY, 8, 0, Math.PI * 2);
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
    const x = event.clientX - rect.left - canvasRef.value.width / 2;
    const y = canvasRef.value.height / 2 - (event.clientY - rect.top);
    
    // Check if we're clicking on an existing point or polygon center
    for (let i = 0; i < props.polygons.length; i++) {
      const polygon = props.polygons[i];
      
      // Check center point
      const center = getPolygonCenter(polygon.points);
      if (Math.abs(center.x - x) < 10 && Math.abs(center.y - y) < 10) {
        return; // Don't add a new point if clicking on center
      }
      
      // Check polygon points
      for (let j = 0; j < polygon.points.length; j++) {
        const point = polygon.points[j];
        if (Math.abs(point.x - x) < 5 && Math.abs(point.y - y) < 5) {
          return; // Don't add a new point if clicking on existing point
        }
      }
    }
    
    // If we're not clicking on an existing point or center, add a new point
    emit('add-point', x, y);
  }
}

function handleMouseMove(event) {
  if (isDragging.value && draggedItem.value) {
    const rect = canvasRef.value.getBoundingClientRect();
    const x = event.clientX - rect.left - canvasRef.value.width / 2;
    const y = canvasRef.value.height / 2 - (event.clientY - rect.top);
    
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
  const x = event.clientX - rect.left - canvasRef.value.width / 2;
  const y = canvasRef.value.height / 2 - (event.clientY - rect.top);
  
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

