<script setup lang="ts">
import Scores from './components/Scores.vue'
import { io } from "socket.io-client";
import { useTemplateRef, onMounted } from 'vue'

const score = useTemplateRef("score");
const socket = io('http://lasergameserver.hugotran.fr:3001');

socket.on('connect', () => {
            console.log('Connecté au serveur WebSocket');
            socket.emit('client_data', {id: 'client'});
        });

socket.on('message', (data) => {
    console.log('Message du serveur:', data);
});

function startGame() {
    socket.emit('start_game');
}

function resetGame() {
    socket.emit('reset_game');
}

socket.on("flags", (data) => {
    console.log('Flags:', data);
});

socket.on("state", (data) => {
    score.value.updateValue(JSON.parse(data))
});

</script>

<template>
  <Scores ref="score" />
    <button @click="startGame">Start game</button>
    <button @click="resetGame">Reset</button>
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
