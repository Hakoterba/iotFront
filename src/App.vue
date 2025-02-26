<template>
  <Scores ref="score" />
  <div class="flex justify-center space-x-4">
    <button class="bg-gradient-to-tr from-blue-700 to-blue-400 rounded-md text-white cursor-pointer px-10 py-1" @click="startGame">Start</button>
    <button class="bg-gradient-to-tr from-red-700 to-red-400 rounded-md text-white cursor-pointer px-10 py-1" @click="resetGame">Reset</button>
  </div>
  <div class="w-full flex justify-center items-center my-10">
    <FlagCanvas ref="flagCanvas"/>
  </div>
</template>

<script>
import Scores from './components/Scores.vue';
import FlagCanvas from '../src/components/FlagCanva.vue';

export default {
  components: {
    Scores,
    FlagCanvas
  },
  data() {
    return {
      socket: null
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
              alert(data.data);
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
