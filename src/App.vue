<template>
  <Scores ref="score" />
  <button @click="startGame">Start game</button>
  <button @click="resetGame">Reset</button>
 
</template>

<script>
import { io } from "socket.io-client";
import Scores from './components/Scores.vue'

export default {
  components: {
    Scores
  },
  data() {
    return {
      stageSize: {
        width: window.innerWidth,
        height: window.innerHeight
      },
      isDragging: false,
      image: null,
      socket: null
    };
  },
  methods: {
    startGame() {
        this.socket.emit('start_game');
    },
    resetGame() {
        this.socket.emit('reset_game');
    }
  },
  mounted() {
    console.log('Connexion au serveur WebSocket');
    
    this.socket = io('http://lasergameserver.hugotran.fr:3001');

    this.socket.on('connect', () => {
                console.log('Connecté au serveur WebSocket');
                this.socket.emit('client_data', {id: 'client'});
            });

    this.socket.on('message', (data) => {
        console.log('Message du serveur:', data);
    });

    this.socket.on("flags", (data) => {
        console.log('Flags:', data);
    });

    this.socket.on("state", (data) => {
      console.log('State:', this.$refs.score);
      
        this.$refs.score.updateValue(JSON.parse(data))
    });
  }
};
</script>
