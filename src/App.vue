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
import { io } from "socket.io-client";
import Scores from './components/Scores.vue'
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
        this.$refs.flagCanvas.setFlags(JSON.parse(data));
        console.log('Flags:', data);
    });

    this.socket.on("state", (data) => {
        this.$refs.flagCanvas.updateFlagsColor(JSON.parse(data).flags);
        this.$refs.score.updateValue(JSON.parse(data))
    });
  }
};

</script>
