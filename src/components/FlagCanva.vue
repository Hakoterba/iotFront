<template>
    <div>
        <h1>Capture de drapeaux</h1>
        <div id="canvasContainer">
            <canvas id="canvas1" ref="canvas1Ref" width="900" height="500"></canvas>
            <canvas id="canvas2" ref="canvas2Ref" width="300" height="500"></canvas>
        </div>
    </div>
</template>
  
  <script setup lang="ts">
  import { onMounted, ref, reactive } from 'vue';
  
  const canvas1Ref = ref<HTMLCanvasElement | null>(null);
  const canvas2Ref = ref<HTMLCanvasElement | null>(null);
  let ctx1: CanvasRenderingContext2D | null = null;
  let ctx2: CanvasRenderingContext2D | null = null;
  
  const state = reactive({
    offsetX: 0,
    offsetY: 0,
    dragging: false,
    draggedImage: null as any,
    imagePositions: [] as { image: HTMLImageElement, x: number, y: number, canvas: string }[],
  });
  
  const flagsData = [
    { id: "567d5781-e49a-4ba1-867c-b53632fe2b7f", socket: "E1DSjMDuE6AXjMRWAAAQ", color: "red" },
    { id: "098d62b8-c68e-4cce-821a-cae81bde0a90", socket: "yBmcFQXN6XMqHaZuAAAS", color: "white" }
  ];
  
  const flagImg = new Image();
  flagImg.src = './src/assets/flag.png';
  
  flagImg.onload = () => {
    let xPos = 50;
    flagsData.forEach(() => {
      state.imagePositions.push({
        image: flagImg,
        x: xPos,
        y: 50,
        canvas: 'canvas2'
      });
      xPos += 90;
    });
    drawImages();
  };
  
  function drawImages(): void {
    if (!ctx1 || !ctx2) return;
  
    ctx1.clearRect(0, 0, canvas1Ref.value!.width, canvas1Ref.value!.height);
    ctx2.clearRect(0, 0, canvas2Ref.value!.width, canvas2Ref.value!.height);
  
    state.imagePositions.forEach((imageObj) => {
      const ctx = imageObj.canvas === 'canvas1' ? ctx1 : ctx2;
      ctx?.drawImage(imageObj.image, imageObj.x, imageObj.y, 150, 150);
    });
  }
  
  function checkCanvas(mouseX: number, mouseY: number) {
    return state.imagePositions.find(imageObj =>
      mouseX >= imageObj.x && mouseX <= imageObj.x + 150 &&
      mouseY >= imageObj.y && mouseY <= imageObj.y + 150
    );
  }
  
  function startDrag(e: MouseEvent): void {
    const mouseX = e.offsetX;
    const mouseY = e.offsetY;
    state.draggedImage = checkCanvas(mouseX, mouseY);
  
    if (state.draggedImage) {
      state.dragging = true;
      state.offsetX = mouseX - state.draggedImage.x;
      state.offsetY = mouseY - state.draggedImage.y;
    }
  }
  
  function dragImage(e: MouseEvent): void {
    if (!state.dragging) return;
  
    const mouseX = e.offsetX;
    const mouseY = e.offsetY;
    state.draggedImage.x = mouseX - state.offsetX;
    state.draggedImage.y = mouseY - state.offsetY;
    drawImages();
  }
  
  function stopDrag(): void {
    state.dragging = false;
    state.draggedImage = null;
    drawImages();
  }
  
  function transferImage(canvasTarget: string): void {
    if (state.draggedImage) {
      state.draggedImage.canvas = canvasTarget;
      drawImages();
    }
  }
  
  onMounted(() => {
    if (!canvas1Ref.value || !canvas2Ref.value) return;
  
    ctx1 = canvas1Ref.value.getContext('2d');
    ctx2 = canvas2Ref.value.getContext('2d');
  
    const addEventListeners = (canvas: HTMLCanvasElement, transferTo: string) => {
      canvas.addEventListener('mousedown', startDrag);
      canvas.addEventListener('mousemove', dragImage);
      canvas.addEventListener('mouseup', stopDrag);
      canvas.addEventListener('mouseleave', () => transferImage(transferTo));
    };
  
    addEventListeners(canvas1Ref.value, 'canvas2');
    addEventListeners(canvas2Ref.value, 'canvas1');
  });
  </script>
  
  <style>
  #canvasContainer {
    width: 1200px;
    height: 500px;
    position: relative;
  }
  
  canvas {
    position: absolute;
    border: 1px solid black;
    background: transparent;
  }
  
  #canvas1 {
    left: 0;
  }
  
  #canvas2 {
    left: 900px;
  }
  </style>
  