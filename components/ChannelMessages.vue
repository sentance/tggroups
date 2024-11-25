<template>
    <div v-if="channel" class="body-messages">
      <h1>{{ channel.name }}</h1>
      <p>Type: {{ channel.type }}</p>
      <p>ID: {{ channel.id }}</p>
  
      <h2>Messages</h2>
      <div v-for="message in channel.messages" :key="message.id" class="message">
        <!-- Render service messages -->
        <div v-if="message.type === 'service'" class="service-message">
          <p><strong>Service Message:</strong> {{ message.action }}</p>
          <p>Actor: {{ message.actor }}</p>
          <p>Title: {{ message.title }}</p>
        </div>
  
        <!-- Render regular messages -->
        <div v-else-if="message.type === 'message'" class="text-message">
          <p><strong>From:</strong> {{ message.from }}</p>
          <p><strong>Date:</strong> {{ formatDate(message.date) }}</p>
          <!-- Render photo if available -->
          <div class="photo-container" v-if="message.photo">
            <img
              :src="`/${message.photo}`"
              :alt="'Photo from message ' + message.id"
              :width="message.width"
              :height="message.height"
              class="message-photo"
            />
          </div>
          <div class="message-text">
            <span
              v-for="(textPart, index) in message.text"
              :key="index"
              :class="getTextClass(textPart)"
              v-html="formatText(textPart)"
            ></span>
          </div>
  
          <div class="reactions" v-if="message.reactions">
            <p><strong>Reactions:</strong></p>
            <div
              v-for="reaction in message.reactions"
              :key="reaction.emoji"
              class="reaction"
            >
              {{ reaction.emoji }} ({{ reaction.count }})
            </div>
          </div>
        </div>
      </div>
    </div>
    <div v-else>
      <p>Loading data...</p>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from "vue";
  
  // Reactive state for the channel data
  const channel = ref(null);
  
  // Fetch JSON data on component mount
  onMounted(async () => {
    try {
      const response = await fetch("/result.json");
      channel.value = await response.json();
      console.log("Loaded channel data:", channel.value); // Debugging
    } catch (error) {
      console.error("Failed to load channel data:", error);
    }
  });
  
  // Helper function to style text
  function getTextClass(textPart) {
    if (typeof textPart === "string") return "";
    if (textPart.type === "italic") return "italic";
    if (textPart.type === "bold") return "bold";
    if (textPart.type === "hashtag") return "hashtag";
    return "";
  }
  
  // Helper function to format text (handling newlines)
  function formatText(textPart) {
    if (typeof textPart === 'string') {
      return textPart.replace(/\n\n/g, '<br><br>').replace(/\n/g, '<br>');
    }
    if (textPart.type === 'text_link') {
      return `<a href="${textPart.href}" target="_blank">${textPart.text}</a>`;
    }
    if (textPart.type === 'hashtag') {
      return `<a href="https://www.example.com/hashtag/${textPart.text}" target="_blank" class="hashtag">${textPart.text}</a>`;
    }
    return textPart.text || textPart;
  }
  
  // Helper function to format date
  function formatDate(dateString) {
    return new Date(dateString).toLocaleString();
  }
  </script>
  
  <style scoped>
  .body-messages {
    padding-top: 64px;
    width: 480px;
    margin: 0 auto;
  }
  .message {
    margin-bottom: 20px;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
  .service-message {
    background-color: #f9f9f9;
    padding: 10px;
    border-left: 4px solid #ff9800;
  }
  .text-message {
    font: 12px / 18px 'Open Sans', "Lucida Grande", "Lucida Sans Unicode", Arial, Helvetica, Verdana, sans-serif;
    background-color: #eef;
    padding: 10px;
    border-left: 4px solid #2196f3;
  }
  .message-text {
    margin-top: 10px;
    white-space: pre-wrap; /* Preserve whitespace formatting */
  }
  .photo-container {
    margin-top: 10px;
  }
  .message-photo {
    max-width: 100%;
    height: auto;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
  .italic {
    font-style: italic;
  }
  .bold {
    font-weight: bold;
  }
  .hashtag {
    color: #007bff;
    text-decoration: none;
  }
  .hashtag:hover {
    text-decoration: underline;
  }
  .reaction {
    display: inline-block;
    margin-right: 10px;
  }
  </style>
  