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
  <div>
    <h1>Lasergame</h1>
    <button @click="startGame">Start game</button>
    <button @click="resetGame">Reset</button>
    <a href="https://vuejs.org/" target="_blank">
      <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
    </a>
  </div>
  <Scores ref="score"/>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
