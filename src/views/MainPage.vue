<template>
  <div class="container">
    <div class="left">
      <!-- Text Above Chat Box -->
      <h2 class="chat-heading">Any questions about my CV? Feel free to ask! :D</h2>

      <!-- Messages Display Box -->
      <div class="chat-box" ref="chatBox">
        <p
          v-for="(message, index) in messages"
          :key="index"
          class="message"
          :class="{ 'message-left': index % 2 === 0, 'message-right': index % 2 !== 0 }"
        >
          {{ message }}
        </p>
      </div>

      <!-- Input Field and Send Button -->
      <div class="input-container">
        <input
          type="text"
          class="input-box"
          v-model="newMessage"
          placeholder="Enter text here..."
          @keyup.enter="!isSendDisabled && sendMessage()"
        />
        <button class="send-button" @click="sendMessage" :disabled="isSendDisabled">Send</button>
      </div>
    </div>

    <div class="right">
      <img src="/CV.png" alt="CV Image" class="image" />
    </div>

    <!-- Pop-Up Modal (Shown when messages reach 6) -->
    <div v-if="showPopup" class="popup-overlay">
      <div class="popup-box">
        <h3>You've sent 6 messages!</h3>
        <p>Keep the conversation going!</p>
        <button class="close-button" @click="closePopup">Close</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, nextTick } from 'vue'

// Reactive array to store messages
const messages = ref(['Type in a question that you want to be answered!'])
const newMessage = ref('')
const chatBox = ref(null)
const showPopup = ref(false) // Controls the pop-up visibility
const isSendDisabled = computed(() => newMessage.value.trim() === '')

// Function to add a new message when "Send" is clicked
const sendMessage = () => {
  if (!isSendDisabled.value) {
    messages.value.push(newMessage.value) // Add message to array
    newMessage.value = '' // Clear input field

    // Auto-scroll to the latest message after Vue updates the DOM
    nextTick(() => {
      if (chatBox.value) {
        chatBox.value.scrollTop = chatBox.value.scrollHeight
      }
    })

    // Show popup when messages reach 6
    if (messages.value.length >= 6) {
      showPopup.value = true
    }
  }
}

// Function to close the pop-up
const closePopup = () => {
  showPopup.value = false
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html,
body,
#app {
  width: 200%;
  height: 100%;
  overflow: hidden;
}

.container {
  display: flex;
  width: 200%;
  height: 120%;
  gap: 20px;
}

.left {
  flex: 1;
  background-color: #a4e1f3; /* Light blue */
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  padding: 20px;
  border-radius: 2%;
}

/* Heading Above Chat Box */
.chat-heading {
  text-align: center;
  font-size: 22px;
  font-weight: bold;
  margin-bottom: 10px;
  color: black;
  font-family: cursive;
}

/* Chat box for displaying messages */
.chat-box {
  width: 100%;
  height: 400px;
  background-color: #d0e8ff;
  border-radius: 10px;
  padding: 10px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 5px;
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  flex-grow: 1;
  min-height: 200px;
}

/* Individual messages */
.message {
  background-color: white;
  padding: 8px;
  border-radius: 5px;
  max-width: 60%;
  word-wrap: break-word;
  overflow-wrap: break-word;
  white-space: pre-wrap;
  color: black;
}

/* Messages on the left (Even index) */
.message-left {
  align-self: flex-start;
  background-color: #ffffff;
  text-align: left;
  padding: 10px;
  border-radius: 10px;
  max-width: 60%;
}

/* Messages on the right (Odd index) */
.message-right {
  align-self: flex-end;
  background-color: #c3e6ff;
  text-align: right;
  padding: 10px;
  border-radius: 10px;
  max-width: 60%;
}

/* Input and button container */
.input-container {
  display: flex;
  justify-content: space-between;
  width: 100%;
  padding-top: 10px;
}

/* Input styling */
.input-box {
  flex: 1;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #000;
  border-radius: 5px;
  outline: none;
  background-color: white;
}

/* Default send button styling */
.send-button {
  margin-left: 10px;
  padding: 10px 15px;
  font-size: 16px;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
  cursor: pointer;
  transition:
    background 0.3s,
    opacity 0.3s;
}

/* Hover effect for active button */
.send-button:hover {
  background-color: #0056b3;
}

/* Disabled button styling */
.send-button:disabled {
  background-color: #b0b0b0; /* Gray background */
  cursor: not-allowed; /* Shows "not allowed" cursor */
  opacity: 0.6; /* Makes the button look faded */
}

/* Prevent hover effect on disabled button */
.send-button:disabled:hover {
  background-color: #b0b0b0;
}

.right {
  flex: 1;
  display: flex;
  border-radius: 2%;
}

.image {
  width: 100%;
  height: 100%;
  border-radius: 2%;
}

/* === POP-UP STYLING === */

/* Pop-up background overlay */
.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.5); /* Transparent black background */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  color: #000;
}

/* Pop-up box */
.popup-box {
  background: white;
  padding: 20px;
  width: 300px;
  text-align: center;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
}

/* Close button */
.close-button {
  margin-top: 10px;
  padding: 8px 12px;
  font-size: 16px;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
  cursor: pointer;
}

.close-button:hover {
  background-color: #0056b3;
}
</style>
