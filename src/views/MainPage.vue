<template>
  <div class="container">
    <div class="left">
      <!-- Text Above Chat Box -->
      <h2 class="chat-heading">Any questions about my CV? Feel free to ask! 😊😊</h2>

      <!-- Recruiter Name Input -->
      <div class="recruiter-input-container">
        <input
          type="text"
          class="recruiter-input-box"
          v-model="recruiterName"
          placeholder="Enter your name (Recruiter)"
        />
      </div>

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

    <!-- Button Group Container (Separated for clarity) -->
    <div class="button-group-container">
      <div class="button-container">
        <!-- Gmail Button -->
        <button class="action-button" @click="openGmail">
          <img src="/gmail.png" alt="Gmail" class="button-icon" />
        </button>

        <!-- LinkedIn Button -->
        <button class="action-button" @click="openLinkedin">
          <img src="/linkedin.png" alt="LinkedIn" class="button-icon" />
        </button>

        <!-- Star Button -->
        <button class="action-button" @click="openPopup">
          <img src="/star.png" alt="Star" class="button-icon" />
        </button>
      </div>
    </div>

    <!-- Pop-Up Modal (Shown when messages reach 6) -->
    <div v-if="showPopup" class="popup-overlay">
      <div class="popup-box">
        <!-- Fun Message -->
        <h3 class="popup-title">
          Take it easy on the messages, they ain’t free
          <span class="emoji">😅</span>! I’d love to hear your thoughts on the website!
        </h3>

        <!-- Name/Company Input -->
        <div class="input-group">
          <label for="name">Your Name (or Company Name):</label>
          <input type="text" id="name" v-model="userName" placeholder="Enter your name..." />
        </div>

        <!-- Website Feedback Input -->
        <div class="input-group">
          <label for="feedback">Your thoughts on the website:</label>
          <textarea
            id="feedback"
            v-model="userFeedback"
            placeholder="Write your opinion here..."
          ></textarea>
        </div>

        <!-- Buttons -->
        <div class="popup-buttons">
          <button class="cancel-button" @click="closePopup">Cancel</button>
          <button class="submit-button" @click="submitFeedback">Submit</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, nextTick } from 'vue'

// Reactive array to store messages
const messages = ref(['Type in a question that you want the answer to!'])
const newMessage = ref('')
const recruiterName = ref('') // New recruiter name input
const chatBox = ref(null)
const showPopup = ref(false)
const userName = ref('')
const userFeedback = ref('')
const isSendDisabled = computed(() => newMessage.value.trim() === '')
const apiUrl = import.meta.env.VITE_API_BASE_URL
// console.log('API Base URL:', apiUrl)

// Function to add a new message when "Send" is clicked
const openAiApiKey = import.meta.env.VITE_OPEN_AI_API_KEY // Get API Key from .env
const openAiApiUrl = 'https://api.openai.com/v1/chat/completions' // OpenAI API URL
const textPredefined = import.meta.env.VITE_CV_PREDEFINED_TEXT
const hasSubmittedFeedback = ref(false) // Track if feedback was submitted

console.log('What are you looking for?')

const openGmail = () => {
  window.open(`https://mail.google.com/mail/u/0/?pli=1#inbox?compose=new`, '_blank')
}

const openPopup = () => {
  if (!hasSubmittedFeedback.value) {
    showPopup.value = true
  } else {
    alert('You have already rated the app! 😊')
  }
}

const openLinkedin = () => {
  alert(
    `I don't have a Linkeding profile right now but i will be making one in the near future! :)`,
  )
}

const sendMessage = async () => {
  if (!isSendDisabled.value) {
    // console.log(textPredefined)
    const messageText = newMessage.value.trim()
    if (recruiterName.value == '') {
      recruiterName.value = 'None'
    }
    // Construct request payload for API Gateway
    const payload = {
      recruiter_name: recruiterName.value,
      recruiter_message: messageText,
    }

    // if (recruiterName.value == 'None') {
    //   recruiterName.value = null
    // }
    try {
      // Send message to API Gateway
      const response = await fetch(`${apiUrl}/recruiterMessages`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(payload),
      })

      const responseData = await response.json()

      if (response.ok) {
        // console.log('Message sent successfully:', responseData)

        // Add the new message to the messages array
        messages.value.push(messageText)
        newMessage.value = '' // Clear input field

        // Auto-scroll to the latest message after Vue updates the DOM
        nextTick(() => {
          if (chatBox.value) {
            chatBox.value.scrollTop = chatBox.value.scrollHeight
          }
        })

        // Show popup when messages reach 6
        if (messages.value.length >= 6 && hasSubmittedFeedback.value == false) {
          showPopup.value = true
        }

        // **Send the message to OpenAI API**
        await getOpenAiResponse(messageText)
      } else {
        console.error('Failed to send message:', responseData)
        alert('Failed to send message. Please try again.')
      }
    } catch (error) {
      console.error('Error sending message:', error)
      alert('An error occurred while sending the message.')
    }
  }
}

const getOpenAiResponse = async (userMessage) => {
  try {
    const openAiPayload = {
      model: 'gpt-3.5-turbo', // Model choice
      messages: [
        {
          role: 'user',
          content:
            `If the question that you recieve is just one word or something like saying thanks, act human. YOU HAVE TO UNDERSTAND, YOU ARE POSING AS ME AND YOU SHOULD NEVER TALK LIKE SOMEONE ELSE. YOUR ANSWERS SHOULD ONLY BE BASED ON THAT ROLE, ROLE OF BEING ME. YOU SHOULDNT BE ASKING ANY QUESTIONS` +
            `Dont just be saying random stuff thats not in the CV that i provided you with, also try to act like a reall person and make the conversation feel like it is natural` +
            textPredefined +
            userMessage,
        },
      ],
      temperature: 0.7,
    }
    // console.log(textPredefined)
    const openAiResponse = await fetch(openAiApiUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${openAiApiKey}`, // Add API Key
      },
      body: JSON.stringify(openAiPayload),
    })

    const openAiData = await openAiResponse.json()

    if (openAiResponse.ok) {
      const aiMessage = openAiData.choices[0].message.content
      // console.log('ChatGPT Response:', aiMessage)

      // **Add AI response as a normal message**
      messages.value.push(aiMessage)

      // Auto-scroll to latest message
      nextTick(() => {
        if (chatBox.value) {
          chatBox.value.scrollTop = chatBox.value.scrollHeight
        }
      })
    } else {
      console.error('Failed to get ChatGPT response:', openAiData)
    }
  } catch (error) {
    console.error('Error calling OpenAI API:', error)
  }
}

const submitFeedback = async () => {
  if (!userName.value.trim() || !userFeedback.value.trim()) {
    alert('Please fill in both fields before submitting!')
    return
  }

  // Construct request payload
  const payload = {
    name: userName.value.trim(),
    opinion: userFeedback.value.trim(),
  }

  try {
    // Send POST request to /newWebsiteRating
    const response = await fetch(`${apiUrl}/newWebsiteRating`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(payload),
    })

    const responseData = await response.json()

    if (response.ok) {
      alert('Thanks for your feedback! 😊')
      userName.value = ''
      userFeedback.value = ''
      closePopup()
      hasSubmittedFeedback.value = true
    } else {
      console.error('Failed to submit feedback:', responseData)
      alert('Failed to submit feedback. Please try again.')
    }
  } catch (error) {
    console.error('Error submitting feedback:', error)
    alert('An error occurred while submitting feedback.')
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

/* 1) Remove default scrollbars from the page */
html,
body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow: hidden; /* No scrollbars */
}

/* 2) #app should fill the browser viewport */
#app {
  width: 100%;
  height: 100%;
  position: relative;
}

/* 3) Container that splits into left/right sections */
.container {
  display: flex;
  flex-direction: row;
  width: 100%;
  height: 100%;
  gap: 20px;
  position: relative;
  align-items: flex-start;
  /* Remove 200%/120% from earlier code! */
}

/* 4) Left Column: Chat UI */
.left {
  flex: 8; /* Fill available space */
  background-color: #a4e1f3; /* Light blue */
  display: flex;
  flex-direction: column;
  justify-content: flex-start; /* Moves content up */
  padding: 20px;
  border-radius: 2%;
  overflow: hidden;
}

/* Chat heading */
.chat-heading {
  text-align: center;
  font-size: 22px;
  font-weight: bold;
  margin-bottom: 10px;
  color: black;
  font-family: cursive;
}

/* Chat box for messages */
.chat-box {
  width: 100%;
  height: 800px; /* Fixed height for chat area */
  background-color: #d0e8ff;
  border-radius: 10px;
  padding: 10px;
  overflow-y: auto; /* Only vertical scroll if needed */
  display: flex;
  flex-direction: column;
  gap: 5px;
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  flex-grow: 1; /* Expand to fill leftover vertical space */
  min-height: 200px;
}

/* Right Column: CV image */
.right {
  flex: 8; /* Fill available space */
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 2%;
  overflow: hidden;
  width: 1500px;
}

/* The CV image */
.image {
  width: 100%;
  height: 100%;
  object-fit: cover; /* Keep ratio, cover area */
  border-radius: 2%;
}

/* Input container for chat text + send button */
.input-container {
  display: flex;
  justify-content: space-between;
  width: 100%;
  padding-top: 10px;
}

/* Recruiter text input */
.recruiter-input-box,
.input-box {
  width: 80%;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #000;
  border-radius: 5px;
  outline: none;
  background-color: white;
  text-align: center;
}

/* Send button */
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
.send-button:hover {
  background-color: #0056b3;
}
.send-button:disabled {
  background-color: #b0b0b0;
  cursor: not-allowed;
  opacity: 0.6;
}
.send-button:disabled:hover {
  background-color: #b0b0b0;
}

/* Outer Button Group Container */
.button-group-container {
  width: 100%;
  display: flex;
  justify-content: flex-start; /* Align to the left */
  position: absolute;
  bottom: 20px; /* Adjust spacing from bottom */
  left: 20px; /* Push it slightly away from the left edge */
}

/* Button Container (Gmail, LinkedIn, Star) */
.button-container {
  display: flex;
  gap: 15px; /* Increased gap for better spacing */
  background: rgba(255, 255, 255, 0.4); /* Semi-transparent background */
  padding: 10px 20px;
  border-radius: 12px; /* Rounded edges for a sleek look */
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.2); /* Soft shadow for depth */
}

/* Circular Icon Buttons */
.action-button {
  width: 50px;
  height: 50px;
  border: none;
  background: transparent;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: transform 0.2s ease-in-out;
}
.action-button:hover {
  transform: scale(1.2);
}

/* Icons inside the buttons */
.button-icon {
  max-width: 80%;
  max-height: 80%;
  object-fit: contain;
  border-radius: 50%;
}

/* Popup Overlay */
.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  color: #000;
}

/* Popup Box */
.popup-box {
  background: white;
  padding: 20px;
  width: 350px;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
  text-align: center;
}

/* Fun Message Title */
.popup-title {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 15px;
}

/* Emoji Style */
.emoji {
  font-size: 20px;
}

/* Input Group in Popup */
.input-group {
  text-align: left;
  margin-bottom: 10px;
}
.input-group label {
  display: block;
  font-size: 14px;
  font-weight: bold;
  margin-bottom: 5px;
}
.input-group input,
.input-group textarea {
  width: 100%;
  padding: 8px;
  font-size: 14px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
.input-group textarea {
  height: 80px;
  resize: none;
}

/* Popup Buttons */
.popup-buttons {
  display: flex;
  justify-content: space-between;
  margin-top: 15px;
}
.cancel-button,
.submit-button {
  padding: 8px 12px;
  font-size: 14px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background 0.3s ease-in-out;
}
.cancel-button {
  background: #ccc;
}
.submit-button {
  background: #007bff;
  color: white;
}
.cancel-button:hover {
  background: #b0b0b0;
}
.submit-button:hover {
  background: #0056b3;
}

/* Messages (Left/Right) */
.message {
  background-color: white;
  padding: 8px;
  border-radius: 5px;
  max-width: 60%;
  word-wrap: break-word;
  white-space: pre-wrap;
  color: black;
}
.message-left {
  align-self: flex-start;
  background-color: #ffffff;
  text-align: left;
  padding: 10px;
  border-radius: 10px;
}
.message-right {
  align-self: flex-end;
  background-color: #c3e6ff;
  text-align: right;
  padding: 10px;
  border-radius: 10px;
}

.recruiter-input-container {
  display: flex;
  justify-content: center; /* Centers horizontally */
  width: 100%; /* Ensures it takes full width of the parent */
  margin-top: 10px; /* Adjust if needed */
}
</style>
