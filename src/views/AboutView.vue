<template>
  <div>
    <canvas ref="drawingCanvas" width="500" height="500"></canvas>
    <br />
    <input type="color" v-model="color" /> <!-- Color picker -->
    <button @click="clearCanvas">Clear Canvas</button>
    <button @click="increaseBrushSize">Increase Brush Size</button>
    <button @click="decreaseBrushSize">Decrease Brush Size</button>
    <p>Brush Size: {{ gridSize }}</p> <!-- Display the current brush size -->
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const drawingCanvas = ref(null);
const ctx = ref(null);
const color = ref('#000000'); // Default drawing color (black)
let painting = ref(false); // Track if the user is currently painting

// Size of each grid cell (brush size)
const gridSize = ref(10);  // Start with a brush size of 10px

// Set up initial canvas and context
const setupCanvas = () => {
  ctx.value = drawingCanvas.value.getContext('2d');
  ctx.value.lineWidth = gridSize.value; // Set the initial brush size
  ctx.value.lineCap = 'square';  // Make lines square (helps with grid alignment)
  ctx.value.strokeStyle = color.value; // Set the drawing color
};

// Convert mouse position to grid coordinates based on brush size
const getGridCoordinates = (x, y) => {
  return {
    x: Math.floor(x / gridSize.value) * gridSize.value, // Snap to nearest grid line
    y: Math.floor(y / gridSize.value) * gridSize.value
  };
};

// Handle mouse down (start painting)
const startPainting = (event) => {
  painting.value = true;
  const { x, y } = getGridCoordinates(event.offsetX, event.offsetY);
  ctx.value.beginPath();
  ctx.value.moveTo(x, y);
  ctx.value.lineTo(x, y); // Draw immediately at the start
  ctx.value.stroke();
};

// Handle mouse move (drawing while painting)
const draw = (event) => {
  if (!painting.value) return;
  const { x, y } = getGridCoordinates(event.offsetX, event.offsetY);
  ctx.value.lineTo(x, y);
  ctx.value.stroke();
};

// Handle mouse up (stop painting)
const stopPainting = () => {
  painting.value = false;
};

// Clear the canvas
const clearCanvas = () => {
  ctx.value.clearRect(0, 0, drawingCanvas.value.width, drawingCanvas.value.height);
};

// Increase brush size by 2
const increaseBrushSize = () => {
  gridSize.value += 2; // Increase by 2 pixels
  updateBrushSize();
};

// Decrease brush size by 2 (minimum size of 2px)
const decreaseBrushSize = () => {
  if (gridSize.value > 2) {
    gridSize.value -= 2; // Decrease by 2 pixels
    updateBrushSize();
  }
};

// Update brush size in the canvas context
const updateBrushSize = () => {
  ctx.value.lineWidth = gridSize.value; // Set the new brush size
};

onMounted(() => {
  setupCanvas();
  const canvasElement = drawingCanvas.value;

  canvasElement.addEventListener('mousedown', startPainting);
  canvasElement.addEventListener('mousemove', draw);
  canvasElement.addEventListener('mouseup', stopPainting);
  canvasElement.addEventListener('mouseleave', stopPainting);
});
</script>

<style scoped>
canvas {
  border: 1px solid black;
  cursor: crosshair;
}
</style>
