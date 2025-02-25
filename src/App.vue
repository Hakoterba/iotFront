<template>
  <v-stage ref="stage" :config="stageSize">
    <v-layer ref="layer">
      <v-image
        @dragstart="handleDragStart"
        @dragend="handleDragEnd"
        :config="{
          image: image,
          x: 50,
          y: 50,
          draggable: true,
          width: 150,
          height: 150
        }"
      />
    </v-layer>
  </v-stage>
</template>

<script>
import { ref, onMounted } from "vue";

export default {
  data() {
    return {
      stageSize: {
        width: window.innerWidth,
        height: window.innerHeight
      },
      isDragging: false,
      image: null
    };
  },
  methods: {
    handleDragStart() {
      this.isDragging = true;
    },
    handleDragEnd(e) {
      this.isDragging = false;
      // Optionnel: tu peux récupérer les nouvelles coordonnées
      console.log('Position après déplacement:', e.target.x(), e.target.y());
    },
    loadImage() {
      const img = new Image();
      img.src = "https://via.placeholder.com/150"; // Remplace par ton URL d'image
      img.onload = () => {
        this.image = img;
      };
    }
  },
  mounted() {
    this.loadImage();
  }
};
</script>
