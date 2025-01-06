<script setup>
import { ref, onMounted, reactive, watch } from 'vue';
import html2canvas from 'html2canvas'; // Import the html2canvas library

const canvas1 = ref(null);
const canvas2 = ref(null);

const colors = reactive({
  bgColor: '',
  paintColor: '',
});

const switchColors = reactive({
  bgColor: '',
  paintColor: '',
});

const ALGORITHM = {
  random_box: 'RANDOM_BOX',
  contagious: 'CONTANGIOUS',
  random: 'RANDOM',
  wave: 'WAVE',
  sine: 'SINE',
  compound: 'COMPOUND'
}

const chooseAlgorithm = ref(ALGORITHM.random_box)

const CONTAGIOUS_CONFIG = reactive({
  base: 0.1,
  samey: 0.2,
  samex: 0.2,
  diag: 0.4,
  quota: 200
})

const COMPOUND_CONFIG = reactive({
  xMark: false,
  yMark: false
})

const WAVE_CONFIG = reactive({
  amplitude: 2,
  frequency: 0.5,
  phaseShift: 1
})

const box = ref([])

const SINE_CONFIG = reactive({
  A: 10,
  B: 0.2,
  C: 10,
  D: 50
})


function switchFunction() {
  switchColors.bgColor = colors.paintColor;
  switchColors.paintColor = colors.bgColor;
}

// Function to export the content of the div containing multiple canvases to PNG
const exportToPNG = () => {
  const container = document.getElementById('bg'); // Select div by id
  
  html2canvas(container).then((canvas) => {
    // Convert the captured canvas to a PNG data URL
    const dataURL = canvas.toDataURL('image/png');

    // Create a temporary link element to trigger the download
    const link = document.createElement('a');
    link.href = dataURL;
    link.download = 'canvas-content.png'; // Set the name of the downloaded file
    link.click(); // Trigger the download
  });
};


// Helper function to generate a random hex color
const getRandomColor = () => {
  const randomColor = Math.floor(Math.random() * 16777215).toString(16); // Generate a random color hex code
  return `#${randomColor.padStart(6, '0')}`;
};


const MAX_WIDTH = 500;

// Function to draw on a canvas
const drawCanvas = (canvas, color = undefined, box = undefined) => {
  const ctx = canvas.getContext('2d');

  // Generate two random colors
 let bgColor, paintColor, boxes

  if (color === undefined) {
    bgColor = getRandomColor(); // Background color
    paintColor = getRandomColor(); // Paint color
  }else{
    bgColor = color.bgColor; // Background color
    paintColor = color.paintColor; // Paint color
  }
  if (box === undefined) {
    switch (chooseAlgorithm.value) {
      case ALGORITHM.random_box:
        boxes = randomBox()
        break;
      case ALGORITHM.contagious:
        boxes = contagious(CONTAGIOUS_CONFIG.base, CONTAGIOUS_CONFIG.quota)
        break;
      case ALGORITHM.random:
        boxes = random(0.5)
        break;
      case ALGORITHM.wave:
        boxes = applyWavePattern()
        break;
      case ALGORITHM.sine:
        boxes = sine()
        break;
      case ALGORITHM.compound:
        boxes = compound()
        break;
      default:
        boxes = randomBox()
    }
  }else{
    boxes = box
  }

  // Set the background color
  ctx.fillStyle = bgColor;
  ctx.fillRect(0, 0, MAX_WIDTH, MAX_WIDTH); // Fill the entire canvas with the background color


  // Set the paint color
  for (let i = 0; i < boxes.length; i++) {
    ctx.fillStyle = paintColor;
    ctx.fillRect(boxes[i].x, boxes[i].y, boxes[i].width, boxes[i].height); // Draw the random rectangle on top of the background
  }

  return { bgColor, paintColor, boxes}; // Return colors for later use
};



// Function to create new canvases and copy content from the original canvases
const duplicateCanvas = (sourceCanvas, targetContainer) => {
  // Create a new canvas element
  const newCanvas = document.createElement('canvas');
  newCanvas.width = sourceCanvas.width;
  newCanvas.height = sourceCanvas.height;
  
  // Append the new canvas to the target container (DOM)
  targetContainer.appendChild(newCanvas);
  
  // Get the context of the new canvas and copy content
  const srcCtx = sourceCanvas.getContext('2d');
  const newCtx = newCanvas.getContext('2d');
  
  // Copy the content of the source canvas to the new canvas
  newCtx.drawImage(sourceCanvas, 0, 0);
};


function on_create(){
 // Draw on the first canvas
 const obj = drawCanvas(canvas1.value);
  colors.bgColor = obj.bgColor; colors.paintColor = obj.paintColor;
  box.value = obj.boxes;
  switchFunction();
  drawCanvas(canvas2.value, switchColors, box.value);


  // Duplicate the content from the original canvases to new ones without adding extra refs
  const container = document.getElementById('canvas-container'); // Select div by id

  duplicateCanvas(canvas2.value, container);
  duplicateCanvas(canvas1.value, container);


  const bg = document.getElementById('bg'); // Select div by id

  let cols = 12;
  for (let i = 0; i < 48; i++) {
    if ((Math.floor(i*2/cols)) % 2 === 0) {
      duplicateCanvas(canvas1.value, bg);
      duplicateCanvas(canvas2.value, bg);
    } else {
      duplicateCanvas(canvas2.value, bg);
      duplicateCanvas(canvas1.value, bg);
    } 
  }

  // Copy the result from the first canvas to the second canvas
  // copyCanvasContent(canvas1.value, canvas2.value);
}



function applyWavePattern() {
  let boxes = [];

  // Define wave parameters
  const amplitude = WAVE_CONFIG.amplitude; // Amplitude of the wave
  const frequency = WAVE_CONFIG.frequency; // Frequency of the wave
  const phaseShift = Math.PI * WAVE_CONFIG.phaseShift; // Phase shift of the wave

  // Loop through a grid of points (100 x 100)
  for (let i = 0; i < (MAX_WIDTH/4); i++) {
    for (let k = 0; k < (MAX_WIDTH/4); k++) {
      const temp = {};
      let x = i*4;
      let y = k*4;

      // Apply sine, cosine, and tangent functions to x and y coordinates
      const sineWaveX = Math.sin(x * frequency + phaseShift) * amplitude;
      const cosineWaveY = Math.cos(y * frequency + phaseShift) * amplitude;
      const tangentWaveX = Math.tan(x * frequency / 10) * amplitude; // We divide by 10 to avoid extreme values
      const tangentWaveY = Math.tan(y * frequency / 10) * amplitude;

      // Calculate the new x and y based on the wave functions
      temp.x = x + sineWaveX + tangentWaveX;
      temp.y = y + cosineWaveY + tangentWaveY;

      // Assign fixed width and height
      temp.width = 4;
      temp.height = 4;

      boxes.push(temp);
    }
  }

  return boxes;
}

function sine(){
  let boxes = [];

  // Define wave parameters
  const A = SINE_CONFIG.A ; // Amplitude of the wave
  const B = SINE_CONFIG.B ; // Frequency of the wave
  const C = SINE_CONFIG.C ; // Phase shift of the wave
  const D = SINE_CONFIG.D ; // Vertical shift of the wave


  // Loop through a grid of points (100 x 100)
  for (let i = 0; i < MAX_WIDTH; i++) {
      const temp = {};
      let x = i;

      // Apply sine, cosine, and tangent functions to x and y coordinates
      let y = A*Math.sin(B*x + C) + D

      // Calculate the new x and y based on the wave functions
      temp.x = x 
      temp.y = y

      // Assign fixed width and height
      temp.width = 4;
      temp.height = 4;

      boxes.push(temp);
  }

  return boxes;
}

function refresh(){
  const canvasContainer = document.getElementById('canvas-container');
  const children = canvasContainer.children;
  // Remove all children except the first two
  for (let i = children.length - 1; i >= 2; i--) {
    canvasContainer.removeChild(children[i]);
  }

  const bgDiv = document.getElementById('bg');
  bgDiv.innerHTML = ''; // Or use other methods like removeChild or replaceChildren

  on_create();
}


function randomBox(){
  let boxes = []
  const n = Math.floor(Math.random() * MAX_WIDTH) + 1
  for (let i = 0; i < n; i++) {
    const temp = {}
    temp.x = Math.floor(Math.random() * MAX_WIDTH); // Random x-coordinate
    temp.y = Math.floor(Math.random() * MAX_WIDTH); // Random y-coordinate
    temp.width = Math.floor(Math.random() * (MAX_WIDTH - temp.x)); // Random width
    temp.height = Math.floor(Math.random() * (MAX_WIDTH - temp.y)); // Random height
    boxes.push(temp)
  }
  return boxes
}

function random(percent){
  let boxes = []
  // loop 100 times
  for (let y = 0; y < MAX_WIDTH; y++) {
    for (let x = 0; x < MAX_WIDTH; x++) {
    const temp = {}
    temp.x = y 
    temp.y = x
    temp.width = 1
    temp.height = 1

    // random follow to percent
    if (Math.random() < percent) {
      boxes.push(temp)
    }
    }
  }
  return boxes
}

function compound(){
  let boxes = []
  let grid = []
  let w = Math.random() * 100 +10
  let h = w
  let offsetx = Math.random() * 40
  let offsety = Math.random() * 40
  let shift = Math.random() * 4 + 1
  console.log(w,h,offsetx,offsety, shift)
  for (let x = 0; x < MAX_WIDTH; x++) {
    let mark = 0
    for (let y = 0; y < MAX_WIDTH; y++) {
      if (x==0){
        grid.push([])
      }
    const temp = {}
    temp.y = y 
    temp.x = x
    temp.width = 1
    temp.height = 1



    if (x > offsetx+ (COMPOUND_CONFIG.xMark ? mark : 0)  &&  x < offsetx+ w && y < offsety+ h&& y > offsety + (COMPOUND_CONFIG.yMark ? mark : 0)  ){ 
      boxes.push(temp)
      grid[y].push(1)
      mark += shift
    }
    else{
      grid[y].push(0)
    }
  }
  w+=1
}
  return boxes
}


function contagious(percent, quota){
  let boxes = []
  let grid = []
  // loop 100 times
  for (let x = 0; x < MAX_WIDTH; x++) {
    for (let y = 0; y < MAX_WIDTH; y++) {
      if (x==0){
        grid.push([])
      }
    const temp = {}
    temp.y = y 
    temp.x = x
    temp.width = 1
    temp.height = 1

    // increase percent (contagious)
    let p = 0
    if (x > 0 && grid[y][x-1] === 1) {
      p += CONTAGIOUS_CONFIG.samey
    }
    if (y > 0 && grid[y-1][x] === 1) {
      p += CONTAGIOUS_CONFIG.samex
    }
    if (x > 0 && y < 99 && grid[y+1][x-1] === 1) {
      p += CONTAGIOUS_CONFIG.diag
    }
    let is_quota = false
    // random follow to percent
    if (p ==0 && quota > 0) {
      p = percent
      is_quota = true
    }
    if (Math.random() < p ) {
      if (is_quota) {
          quota -= 1
          boxes.push(temp)
          grid[y].push(1)
        }
      else{
        boxes.push(temp)
        grid[y].push(1)
      }
    }
    else{
      grid[y].push(0)
    }
  }
}
  return boxes
}

onMounted(() => {
  on_create();
});
</script>

<template>
  <div class="flex px-14 justify-between">

     <!-- Button to export the content of the div -->
     
     <div class="flex flex-col gap-4">
      0-{{MAX_WIDTH}} boxes
       <div id="canvas-container" class="border-2 border-red-200 w-fit h-fit grid grid-cols-2">
         <canvas ref="canvas1" :width="100" :height="100"></canvas>
         <canvas ref="canvas2" :width="100" :height="100"></canvas>
        </div>
        <button class="bg-teal-200 border border-gray-400 h-fit" @click="refresh">Refresh</button>
        <button class="bg-teal-200 border border-gray-400 h-fit" @click="exportToPNG">Export to PNG</button>
        <select class='border-2 border-teal-500 rounded' v-model="chooseAlgorithm">
          <option :value="ALGORITHM.random_box">Random Box</option>
          <option :value="ALGORITHM.contagious">Contagious</option>
          <option :value="ALGORITHM.random">Random</option>
          <option :value="ALGORITHM.wave">Scotch</option>
          <option :value="ALGORITHM.sine">Sine</option>
          <option :value="ALGORITHM.compound">Compound</option>

        </select>

        <template v-if="chooseAlgorithm === ALGORITHM.contagious">
          <div class='border-2 flex justify-between gap-8 border-teal-500 rounded'>
            <label>Base</label>
            <input type="number" v-model="CONTAGIOUS_CONFIG.base" min="0" max="0.9" step="0.1" />
          </div>
          <div class='border-2 flex justify-between gap-8 border-teal-500 rounded'>
            <label>Same Y</label>
            <input type="number" v-model="CONTAGIOUS_CONFIG.samey" min="0" max="0.9" step="0.1" />
          </div>
          <div class='border-2 flex justify-between gap-8 border-teal-500 rounded'>
            <label>Same X</label>
            <input type="number" v-model="CONTAGIOUS_CONFIG.samex" min="0" max="0.9" step="0.1" />
          </div>
          <div class='border-2 flex justify-between gap-8 border-teal-500 rounded'>
            <label>Diag</label>
            <input type="number" v-model="CONTAGIOUS_CONFIG.diag" min="0" max="0.9" step="0.1" />
          </div>
          <div class='border-2 flex justify-between gap-8 border-teal-500 rounded'>
            <label>Quota</label>
            <input type="number" v-model="CONTAGIOUS_CONFIG.quota" min="0" />
          </div>
        </template>

        <template v-if="chooseAlgorithm === ALGORITHM.wave">
          <div class='border-2 flex justify-between gap-8 border-teal-500 rounded'>
            <label>Amplitude</label>
            <input type="number" v-model="WAVE_CONFIG.amplitude" min="0" max="10" step="0.1" />
          </div>
          <div class='border-2 flex justify-between gap-8 border-teal-500 rounded'>
            <label>Frequency</label>
            <input type="number" v-model="WAVE_CONFIG.frequency" min="0" max="10" step="0.1" />
          </div>
          <div class='border-2 flex justify-between gap-8 border-teal-500 rounded'>
            <label>Phase Shift</label>
            <input type="number" v-model="WAVE_CONFIG.phaseShift" min="0" max="10" step="0.1" />
          </div>
        </template>

        <template v-if="chooseAlgorithm === ALGORITHM.sine">
          <div class='border-2 flex justify-between gap-8 border-teal-500 rounded'>
            <label>A</label>
            <input type="number" v-model="SINE_CONFIG.A" min="0" max="100" step="0.1" />
          </div>
          <div class='border-2 flex justify-between gap-8 border-teal-500 rounded'>
            <label>B</label>
            <input type="number" v-model="SINE_CONFIG.B" min="0" max="10" step="0.1" />
          </div>
          <div class='border-2 flex justify-between gap-8 border-teal-500 rounded'>
            <label>C</label>
            <input type="number" v-model="SINE_CONFIG.C" min="0" max="50" step="0.1" />
          </div>
          <div class='border-2 flex justify-between gap-8 border-teal-500 rounded'>
            <label>D</label>
            <input type="number" v-model="SINE_CONFIG.D" min="0" max="50" step="0.1" />
          </div>
       
        </template>


        <template v-if="chooseAlgorithm === ALGORITHM.compound">
          <div class="flex">
<div>
  <input type="checkbox" v-model="COMPOUND_CONFIG.xMark" />
  <label>X Mark</label>
</div>
 <div>
   <input type="checkbox" v-model="COMPOUND_CONFIG.yMark" />
   <label>Y Mark</label>
 </div>
          </div>
     
        </template>
      </div>
    
    
    <div class="bg-yellow-200 max-w-[80%] grid grid-cols-12 h-fit" id="bg" >
    </div>
  </div>
</template>
