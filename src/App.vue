<template>
  <div class="container mx-auto py-8">
    <h1 class="text-3xl font-bold mb-4">Generate Text Stream</h1>
    <input v-model="prompt" type="text" class="border border-gray-300 px-3 py-2 rounded-md mb-4"
      placeholder="Enter your prompt">
    <button @click="generateText" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded ml-2">
      Generate Text
    </button>
    <div v-if="error" class="text-red-600 mt-4">{{ error }}</div>
  </div>
  <div class="container">
    <div v-if="text.length > 0" class="mt-4 text-justify">
      <div v-html="parsedText"></div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { marked } from 'marked';


const prompt = ref('');
const text = ref([]);
const error = ref('');
const parsedText = ref('');

async function generateText() {
  try {
    const response = await fetch('https://gdsc-upn-jatim-ml-muf7kziviq-as.a.run.app/generate_text_stream', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ prompt: prompt.value }),
    });
    if (!response.ok) {
      throw new Error('Failed to generate text');
    }
    const reader = response.body.getReader();
    let result = '';
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      result += new TextDecoder().decode(value);
      text.value = result.split('\n').filter(Boolean);
      parsedText.value = marked.parse(result);
    }
    prompt.value = '';
  } catch (err) {
    error.value = err.message;
  }
}
</script>
