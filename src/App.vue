<template>
  <Scores ref="score" />
  <div class="flex justify-center space-x-4">
    <button class="bg-gradient-to-tr from-blue-700 to-blue-400 rounded-md text-white cursor-pointer px-10 py-1 z-101" @click="startGame">Start</button>
    <button class="bg-gradient-to-tr from-red-700 to-red-400 rounded-md text-white cursor-pointer px-10 py-1 z-101" @click="resetGame">Reset</button>
  </div>
  <div class="w-full flex justify-center items-center my-10">
    <FlagCanvas ref="flagCanvas"/>
  </div>

  <transition
    enter-active-class="transition-opacity duration-500"
    leave-active-class="transition-opacity duration-500"
    enter-from-class="opacity-0"
    leave-to-class="opacity-0"
  >
    <div v-if="gameOver" :class="['z-101 fixed top-5 left-1/2 transform -translate-x-1/2 px-6 py-3 text-white rounded-lg shadow-lg flex items-center space-x-3', gameOverColor]">
      <span class="font-semibold">{{ gameOverMessage }}</span>
      <button @click="closeAlert" class="text-white hover:text-gray-300 cursor-pointer">X</button>
    </div>
  </transition>   
</template>

<script>
import Scores from './components/Scores.vue';
import FlagCanvas from '../src/components/FlagCanva.vue';
import { confetti } from "@tsparticles/confetti";

export default {
  components: {
    Scores,
    FlagCanvas
  },
  data() {
    return {
      socket: null,
      gameOver: false,
      gameOverMessage: "",
      gameOverColor: "bg-gray-700"
    };
  },
  methods: {
    startGame() {
      this.sendMessage({ id: 'start_game' });
    },
    resetGame() {
      this.sendMessage({ id: 'reset_game' });
    },
    sendMessage(data) {
      if (this.socket && this.socket.readyState === WebSocket.OPEN) {
        this.socket.send(JSON.stringify(data));
      } else {
        console.error('WebSocket non connecté');
      }
    },
    closeAlert() {
      this.gameOver = false;
    },
    launchConfetti() {
      confetti({
        spread: 70,
        particleCount: 100,
        origin: { y: 0 }
      });
    }
  },
  mounted() {
    console.log('Connexion au serveur WebSocket');
    this.socket = new WebSocket('ws://lasergameserver.hugotran.fr:3001');
    
    this.socket.onopen = () => {
      console.log('Connecté au serveur WebSocket');
      this.sendMessage({ id: 'client' });
    };
    
    this.socket.onmessage = (event) => {
      const data = JSON.parse(event.data);
      console.log('Message du serveur:', data);
      
      switch (data.event) {
        case 'message':
          console.log('Serveur:', data.data);
          if (data.data.includes('Game over!')) {
            setTimeout(() => {
              this.gameOverMessage = data.data;
              this.launchConfetti();

              if (data.data.includes("red")) {
                this.gameOverColor = "bg-red-600";
              } else if (data.data.includes("blue")) {
                this.gameOverColor = "bg-blue-600";
              }
              this.gameOver = true;
            }, 2000);
          }
          break;
        case 'flags':
          this.$refs.flagCanvas.setFlags(data.data);
          break;
        case 'state':
          this.$refs.flagCanvas.updateFlagsColor(data.data.flags);
          this.$refs.score.updateValue(data.data);
          break;
      }
    };
    
    this.socket.onclose = () => {
      console.log('Déconnecté du serveur WebSocket');
    };
  }
};
</script>
