<template>
  <div class="space-y-4">
    <div class="relative flex justify-center items-center space-x-4">
      <canvas class="bg-white rounded-lg shadow-xl" id="canvas1" ref="canvas1Ref" width="1280" height="600"></canvas>
      <canvas class="bg-white rounded-lg shadow-xl" id="canvas2" ref="canvas2Ref" width="300" height="600"></canvas>
      <!-- <Icon icon="hugeicons:flag-02" class="text-6xl" /> -->
    </div>
    <input class="cursor-pointer file:cursor-pointer file:mr-4 file:rounded-full file:border-0 file:bg-lime-50 file:px-4 file:py-2 file:text-sm file:font-semibold file:text-lime-700 hover:file:bg-lime-100 dark:file:bg-lime-600 dark:file:text-lime-100 dark:hover:file:bg-lime-500" type="file" name="" id="" @change="on_upload" />
  </div>
</template>
  
  <script setup lang="ts">
  import { Icon } from "@iconify/vue";
  import { onMounted, ref, reactive, watch } from 'vue';
  
  const canvas1Ref = ref<HTMLCanvasElement | null>(null);
  const canvas2Ref = ref<HTMLCanvasElement | null>(null);
  let ctx1: CanvasRenderingContext2D | null = null;
  let ctx2: CanvasRenderingContext2D | null = null;
  
  const state = reactive({
    offsetX: 0,
    offsetY: 0,
    dragging: false,
    draggedImage: null as any,
    imagePositions: [] as { 
      image: HTMLImageElement, 
      x: number, 
      y: number, 
      width: number, 
      height: number, 
      canvas: string
    }[],
  });
  
  const flagsData = ref<any[]>([]);
  function setFlags(flags: any) {
    flagsData.value = flags;
  }

  defineExpose({ setFlags, updateFlagsColor });
  
  const flagImg = new Image();
  flagImg.src = './src/assets/flag.png';
  
  flagImg.onload = () => {
    let xPos = 50;
    flagsData.value.forEach(() => {
      state.imagePositions.push({
        image: flagImg,
        x: xPos,
        y: 50,
        width: 150,
        height: 150,
        canvas: 'canvas2'
      });
      xPos += 90;
    });
    drawImages();
  };

  watch(flagsData, (newFlags, oldFlags) => {
      const newIds = new Set(newFlags.map(flag => flag.id));

      state.imagePositions = state.imagePositions.filter(pos => newIds.has(pos.image.dataset.id));

      newFlags.forEach(flag => {
          const alreadyExists = state.imagePositions.some(pos => pos.image.dataset.id === flag.id);
          if (!alreadyExists) {
              const newImage = new Image();
              newImage.src = flagImg.src;
              newImage.dataset.id = flag.id;

              state.imagePositions.push({
                  image: newImage,
                  x: 50 + state.imagePositions.length * 90,
                  y: 50,
                  width: 150,
                  height: 150,
                  canvas: 'canvas2'
              });
          }
      });

      drawImages();
  }, { deep: true });

  function updateFlagsColor(flags: any) {
  state.imagePositions.forEach((imageObj) => {
    // Chercher le drapeau correspondant dans le tableau flags
    const flag = flags.find(flag => flag.id === imageObj.image.dataset.id);
    if (flag) {
      console.log(flag);
      
      const color = flag.color;
      if (color === "white") return; // Si la couleur est blanche, ne rien faire

      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      if (!ctx) return;

      // Définir les dimensions du canvas
      canvas.width = 150;
      canvas.height = 150;

      // Appliquer la couleur
      ctx.fillStyle = color;
      ctx.fillRect(0, 0, 150, 150); // Appliquer la couleur en tant que fond

      // Mélanger la couleur avec l'image existante
      ctx.globalCompositeOperation = 'destination-atop';
      ctx.drawImage(imageObj.image, 0, 0, 150, 150); // Dessiner l'image par-dessus la couleur

      // Mettre à jour l'image avec la couleur appliquée
      imageObj.image.src = canvas.toDataURL();
    }
  });
  drawImages();
}



  
  function drawImages(): void {
    if (!ctx1 || !ctx2) return;

    ctx1.clearRect(0, 0, canvas1Ref.value!.width, canvas1Ref.value!.height);
    ctx2.clearRect(0, 0, canvas2Ref.value!.width, canvas2Ref.value!.height);

    const backgroundImage = state.imagePositions.find(img => img.canvas === 'canvas1' && img.width === 1280);
    const otherImages = state.imagePositions.filter(img => img !== backgroundImage);

    if (backgroundImage) {
      ctx1?.drawImage(backgroundImage.image, backgroundImage.x, backgroundImage.y, backgroundImage.width, backgroundImage.height);
    }

    otherImages.forEach((imageObj) => {
      const ctx = imageObj.canvas === 'canvas1' ? ctx1 : ctx2;
      ctx?.drawImage(imageObj.image, imageObj.x, imageObj.y, imageObj.width, imageObj.height);
    });
  }
  
  function checkCanvas(mouseX: number, mouseY: number) {
    return state.imagePositions.find(imageObj =>
      mouseX >= imageObj.x && mouseX <= imageObj.x + imageObj.width &&
      mouseY >= imageObj.y && mouseY <= imageObj.y + imageObj.height
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

function on_upload(e: Event) {
  const file = (e.target as HTMLInputElement).files?.[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = (e) => {
    const img = new Image();
    img.src = e.target?.result as string;
    img.onload = () => {
      state.imagePositions.push({
        image: img,
        x: 50,
        y: 50,
        width: 1280,
        height: 600,
        canvas: 'canvas1'
      });
      drawImages();
    };
  };
  reader.readAsDataURL(file);
}
</script>