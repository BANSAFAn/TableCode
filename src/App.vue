<script setup lang="ts">
import { ref, onMounted } from "vue";
import OpenAI from "openai";
import mermaid from "mermaid";

const apiKey = ref("");
const inputCode = ref("");
const output = ref("");
const action = ref("diagram");
const sourceLang = ref("TypeScript");
const targetLang = ref("Java");
const mermaidRef = ref(null);
let openai: OpenAI | null = null;

onMounted(() => {
  mermaid.initialize({ startOnLoad: true });
});

async function processCode() {
  if (!apiKey.value) {
    output.value = "Please enter OpenAI API key.";
    return;
  }
  openai = new OpenAI({ apiKey: apiKey.value, dangerouslyAllowBrowser: true });
  let prompt = "";
  if (action.value === "diagram") {
    prompt = `Generate Mermaid code for a flowchart from this code: ${inputCode.value}`;
  } else if (action.value === "schema") {
    prompt = `Generate Mermaid code for a class diagram from this code: ${inputCode.value}`;
  } else if (action.value === "translate") {
    prompt = `Translate this ${sourceLang.value} code to ${targetLang.value}: ${inputCode.value}`;
  }
  const response = await openai.chat.completions.create({
    model: "gpt-4",
    messages: [{ role: "user", content: prompt }],
  });
  output.value = response.choices[0].message.content;
  if (action.value !== "translate" && mermaidRef.value) {
    mermaid.render("mermaid-diagram", output.value, (svgCode) => {
      mermaidRef.value.innerHTML = svgCode;
    });
  }
}
</script>

<template>
  <div class="min-h-screen bg-gray-100 flex flex-col items-center p-4">
    <h1 class="text-3xl font-bold mb-6">Code Converter</h1>
    <input v-model="apiKey" placeholder="OpenAI API Key" class="mb-4 p-2 border rounded w-80" />
    <textarea v-model="inputCode" placeholder="Enter code" class="mb-4 p-2 border rounded w-80 h-40"></textarea>
    <select v-model="action" class="mb-4 p-2 border rounded w-80">
      <option value="diagram">To Block Diagram</option>
      <option value="schema">To Schema</option>
      <option value="translate">Translate to another language</option>
    </select>
    <div v-if="action === 'translate'" class="flex mb-4">
      <input v-model="sourceLang" placeholder="Source Lang" class="p-2 border rounded mr-2 w-40" />
      <input v-model="targetLang" placeholder="Target Lang" class="p-2 border rounded w-40" />
    </div>
    <button @click="processCode" class="bg-blue-500 text-white p-2 rounded">Process</button>
    <div v-if="output" class="mt-6 w-80">
      <div v-if="action !== 'translate'" ref="mermaid" class="mermaid"></div>
      <pre v-else class="bg-white p-4 border rounded">{{ output }}</pre>
    </div>
  </div>
</template>

<style>
@tailwind base;
@tailwind components;
@tailwind utilities;
</style>