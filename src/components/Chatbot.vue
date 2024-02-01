<template>
  <div class="chatbot">
    <div v-for="(message, index) in messages" :key="index" class="message" :class="{ 'user-message': message.isUser, 'chatbot-message': !message.isUser }">
      {{ message.text }}
    </div>
    <div class="input-container">
      <input v-model="userMessage" @keyup.enter="sendMessage" placeholder="Type a message..." />
    </div>
  </div>
</template>

<script>
import openai from "openai";

export default {
  data() {
    return {
      messages: [],
      userMessage: "",
    };
  },
  methods: {
    async sendMessage() {
      const userMessageObj = { role: 'user', content: this.userMessage };
      this.messages.push({ text: this.userMessage, isUser: true });
      // Call GPT-3.5 API to get a response and append it to messages
      try {
        const openaiInstance = new openai({ apiKey: process.env.VUE_APP_OPENAI_API_KEY, dangerouslyAllowBrowser: true });

        const response = await openaiInstance.chat.completions.create({
          model: "gpt-3.5-turbo-0613", // GPT-3.5 engine
          messages: [userMessageObj],
          max_tokens: 50, // Adjust as needed
        });

        // Append GPT-3.5 response to the chat
        this.messages.push({ text: response.choices[0].text, isUser: false });
      } catch (error) {
        console.error("Error communicating with GPT-3.5:", error);
      }

      // Clear the input field
      this.userMessage = "";
    },
  },
};
</script>

<style scoped>
.chatbot {
  height: 700px;
  width: 900px;
  border: 1px solid #ccc;
  padding: 10px;
  border-radius: 5px;
  overflow-y: scroll;
  max-height: 400px;
  display: flex;
  flex-direction: column;
  border: 1px solid red;
}

.message {
  margin-bottom: 10px;
  padding: 8px;
  border-radius: 5px;
  background-color: #f0f0f0;
}

.user-message {
  align-self: flex-start;
}

.chatbot-message {
  align-self: flex-end;
}

.input-container {
  align-self: flex-end;
  margin-top: auto; /* Push the input-container to the bottom */
}

input {
  width: 700px;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
</style>
