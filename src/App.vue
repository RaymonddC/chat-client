<script setup>
import { io } from 'socket.io-client';
import { onBeforeMount, ref } from 'vue';

const socket = io('http://localhost:3001');

const messages = ref({});
const messageText = ref('');
const joined = ref(false);
const name = ref('');
const typinngDisplay = ref('');

onBeforeMount(() => {
  socket.emit('findAllMessages', {}, (response) => {
    messages.value = response;
  });

  socket.on('message', (message) => {
    messages.value.push(message);
  });

  socket.on('typing', ({ name, isTyping }) => {
    if (isTyping) {
      typinngDisplay.value = `${name} is typing...`;
    } else {
      typinngDisplay.value = '';
    }
  });
});

const join = () => {
  socket.emit('join', { name: name.value }, () => {
    joined.value = true;
  });
};

const sendMessage = () => {
  socket.emit('createMessage', { text: messageText.value }, (response) => {
    messageText.value = '';
  });
};

let timeout;

const emitTyping = () => {
  socket.emit('typing', { isTyping: true });
  timeout = setTimeout(() => {
    socket.emit('typing', { isTyping: false });
  }, 2000);
};
</script>

<template>
  <div class="chat">
    <div v-if="!joined">
      <form @submit.prevent="join">
        <label>What's your name?</label>
        <input v-model="name" />
        <button type="submit">Send</button>
      </form>
    </div>
    <div v-else class="chat-container">
      <div class="message-container">
        <div v-for="message in messages">[{{ message.name }}]: {{ message.text }}</div>
      </div>

      <div v-if="typinngDisplay">{{ typinngDisplay }}</div>

      <hr />

      <div class="message-input">
        <form @submit.prevent="sendMessage">
          <label>Message:</label>
          <input v-model="messageText" @input="emitTyping" />
          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style>
@import './assets/base.css';

.chat {
  padding: 20px;
  height: 100vh;
}

.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.message-container {
  flex: 1;
}
</style>
