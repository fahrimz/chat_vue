<script lang="ts">
import { onMounted, onBeforeUnmount, ref, defineComponent } from 'vue';

type SystemLog = {
  timestamp: Date;
  message: string;
}

type Chat = {
  timestamp: Date;
  message: string;
}

let ws: WebSocket;
const chatListBottomRef = ref<HTMLDivElement | null>(null);

const chats = ref<Chat[]>([]);
const message = ref('');
const systemLog = ref<SystemLog[]>([]);
const connectionStatus = ref<'connected' | 'disconnected' | 'connecting'>('disconnected');
const reconnectionAttempts = ref(0);
const maxReconnectionAttempts = 3;

function log(message: string) {
  console.log(message);
  systemLog.value.push({
    timestamp: new Date(),
    message,
  });
}

function scrollToBottom(delay = 0) {
  setTimeout(() => {
    chatListBottomRef.value?.scrollIntoView({ behavior: 'smooth' });
  }, delay);
}

function reconnect() {
  if (reconnectionAttempts.value < maxReconnectionAttempts) {
    reconnectionAttempts.value++;
    setTimeout(connect, 1000); // Reconnect after 1 second
  } else {
    log(`Failed to reconnect after few attempts. Please try again later.`);
  }
}

function connect() {
  const url = import.meta.env.VITE_WS_URL;
  if (!url) {
    throw new Error("WS_URL is not defined in the environment variables.");
  }

  log(`Connecting to server...`);
  connectionStatus.value = 'connecting';

  ws = new WebSocket(`${url}/ws`);

  ws.onopen = function () {
    log("Connected to server.");
    connectionStatus.value = 'connected';
  };

  ws.onmessage = function (event) {
    chats.value.push({
      timestamp: new Date(),
      message: event.data,
    });

    scrollToBottom(300);
  };

  ws.onclose = function () {
    log("Disconnected from server.");
    if (reconnectionAttempts.value >= maxReconnectionAttempts) {
      connectionStatus.value = 'disconnected';
    }
    reconnect();
  };

  ws.onerror = function (error) {
    console.log("error: " + JSON.stringify(error));
  };
}

function connectManually() {
  reconnectionAttempts.value = 0;
  connect();
}

// Function to send a message
function sendMessage(msg: string) {
  if (msg.trim() === "") {
    return;
  }

  if (!ws || ws.readyState !== WebSocket.OPEN) {

    log("not connected to server.");
    return;
  }

  ws.send(msg);
  message.value = ''; // Clear the input after sending
  chats.value.push({
    timestamp: new Date(),
    message: msg,
  });

  // Scroll to the bottom of the chat list
  scrollToBottom();
}

function clearChat() {
  chats.value = [];
}

export default defineComponent({
  setup() {
    // Set up connection when component is mounted
    onMounted(connect);

    // Cleanup connection when component is unmounted
    onBeforeUnmount(() => {
      if (ws && ws.readyState === WebSocket.OPEN) {
        ws.close();
      }
    });

    return {
      message,
      chats,
      sendMessage,
      systemLog,
      clearChat,
      chatListBottomRef,
      connectionStatus,
      connectManually,
    };
  }
});

</script>

<template>
  <div class="container">
    <div class="systemLog">
      <span class="title">System Log</span>
      <ul class="logList">
        <li v-for="(log, i) in systemLog" :key="i">
          <span>{{ log.timestamp.toLocaleTimeString() }}</span>
          <p>{{ log.message }}</p>
        </li>
      </ul>
      <div class="status" :class="{
        'statusOn': connectionStatus === 'connected',
        'statusOff': connectionStatus === 'disconnected',
        'statusConnecting': connectionStatus === 'connecting'
      }">
        <span class="statusText">{{ connectionStatus }}</span>
        <div v-if="connectionStatus === 'disconnected'" class="btnConnect" @click="connectManually"><span>Connect</span>
        </div>
      </div>
    </div>
    <div class="chatWall">
      <div class="header">
        <h1 class="title">Public Chat</h1>
        <span class="clear" @click="clearChat">Clear chat</span>
      </div>
      <hr />
      <ul class="chatList">
        <p v-if="chats.length === 0">No messages yet. Start chatting!</p>
        <li v-for="(chat, i) in chats" :key="i">
          <span>{{ chat.timestamp.toLocaleTimeString() }}</span>
          <p>{{ chat.message }}</p>
        </li>
        <div ref="chatListBottomRef" />
      </ul>

      <div class="inputContainer">
        <input v-model="message" @keyup.enter="sendMessage(message)" placeholder="Type a message..." class="input" />
        <button class="send" @click="sendMessage(message)">
          Send
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: row;
  height: 80vh;
  gap: 24px;
}

.systemLog {
  background-color: #f0f0f0;
  border-radius: 5px;
  width: 40%;
  height: 100%;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.title {
  font-weight: bold;
  color: black;
}

.systemLog .title {
  padding: 16px 16px 0px 16px;
}

.chatWall {
  background-color: white;
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 10px;
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.container span,
.container p,
.container li {
  color: black;
}

.container ul {
  padding-left: 0;
}

.container li {
  list-style: none;
  font-size: 0.7rem;
  color: rgba(0, 0, 0, 0.7);
}

.systemLog li {
  display: flex;
  flex-direction: row;
  gap: 1rem;
}

.inputContainer {
  display: flex;
  flex-direction: row;
  gap: 10px;
  margin-bottom: 10px;
}

.input {
  width: 100%;
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #ccc;
}

.send {
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #ccc;
  background-color: #007bff;
  color: white;
  cursor: pointer;
}

hr {
  margin: 10px 0;
}

.chatList {
  flex: 1;
  overflow: scroll;
  margin: 1rem 0;
}

.chatList li {
  display: flex;
  flex-direction: row;
  gap: 1rem;
  align-items: center;
}

.chatList span {
  color: rgba(0, 0, 0, 0.7);
}

.chatList p {
  font-size: 0.8rem;
}

.chatWall .header {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
}

.clear {
  color: var(--color-primary) !important;
  cursor: pointer;
}

.logList {
  flex: 1;
  padding: 0 16px !important;
  overflow: scroll;
}

.status {
  font-size: 0.8rem;
  padding: 16px;
  border-bottom-left-radius: 5px;
  border-bottom-right-radius: 5px;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  height: 50px;
}

.statusOn {
  background-color: var(--color-primary);
}

.statusOff {
  background-color: rgb(215, 53, 53);
}

.statusConnecting {
  background-color: #EB5B00;
}

.statusText {
  color: white !important;
  text-transform: capitalize;
}

.btnConnect {
  cursor: pointer;
  padding: 4px 6px;
  background-color: var(--color-primary);
  border-radius: 5px;
  text-align: center;
}

.btnConnect span {
  color: white;
}

@media (max-width: 767px) {
  .systemLog {
    display: none;
  }
}
</style>