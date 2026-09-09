<template>
  <div class="min-h-screen bg-[#0f111a] text-slate-200 font-sans flex flex-col">
    <nav class="w-full border-b border-white/5 bg-[#161926]/50 backdrop-blur-xl px-12 py-6 flex justify-between items-center">
      <div class="flex items-center gap-8">
        <button @click="reset" class="text-[10px] font-black uppercase tracking-[0.2em] text-slate-500 hover:text-white transition-all">
          ← Discard
        </button>
        <div class="h-4 w-px bg-white/10"></div>
        <h1 class="text-sm font-black uppercase tracking-[0.4em] text-white">Riverflow <span class="text-indigo-500">V2 Pro</span></h1>
      </div>
      
      <div class="flex items-center gap-4">
        <div class="flex flex-col items-end mr-2">
          <span class="text-[9px] font-mono text-slate-500 uppercase tracking-widest">OpenRouter API</span>
          <span :class="apiKey ? 'text-green-500' : 'text-rose-500'" class="text-[10px] font-bold uppercase tracking-widest">
            {{ apiKey ? 'Linked' : 'Missing Key' }}
          </span>
        </div>
        <input v-model="apiKey" type="password" placeholder="sk-or-..." class="bg-black/40 border border-white/5 rounded-xl px-4 py-2 text-[10px] w-52 focus:border-indigo-500/50 outline-none transition-all" />
      </div>
    </nav>

    <main class="flex-1 flex flex-col items-center justify-center p-12">
      <div v-if="!originalImage" class="w-full max-w-[900px] animate-in fade-in zoom-in duration-500">
        <div class="text-center mb-10 space-y-2">
          <h2 class="text-4xl font-black text-white tracking-tighter uppercase italic">Neural <span class="text-indigo-500">Chrome</span></h2>
          <p class="text-[10px] text-slate-500 uppercase tracking-[0.3em]">SOTA Image-to-Image Reasoning Engine</p>
        </div>
        
        <label class="relative block w-full aspect-[21/9] bg-[#161926] rounded-[40px] border-2 border-dashed border-white/5 hover:border-indigo-500/40 hover:bg-indigo-500/[0.02] transition-all cursor-pointer group shadow-2xl">
          <div class="absolute inset-0 flex flex-col items-center justify-center pointer-events-none">
            <div class="mb-4 text-3xl opacity-20 group-hover:opacity-100 transition-all text-indigo-500">＋</div>
            <p class="text-sm font-bold text-slate-300 tracking-tight">Click to upload <span class="font-normal text-slate-500 italic">or drag and drop</span></p>
          </div>
          <input type="file" class="hidden" @change="handleFileUpload" accept="image/*" />
        </label>
      </div>

      <div v-else class="w-full max-w-[1400px] grid grid-cols-2 gap-12 animate-in slide-in-from-bottom-8 duration-700">
        <div class="space-y-4">
          <span class="text-[10px] font-black text-slate-500 uppercase tracking-widest ml-4">Source Layer</span>
          <div class="aspect-video bg-black/60 rounded-[32px] overflow-hidden border border-white/5 flex items-center justify-center shadow-2xl">
            <img :src="originalImage" class="max-h-full max-w-full object-contain" />
          </div>
        </div>

        <div class="space-y-4">
          <span class="text-[10px] font-black text-indigo-400 uppercase tracking-widest ml-4">Processed Layer</span>
          <div class="relative aspect-video bg-black/60 rounded-[32px] overflow-hidden border border-indigo-500/20 shadow-2xl flex items-center justify-center">
            <div v-if="isProcessing" class="absolute inset-0 z-30 flex flex-col items-center justify-center bg-[#0f111a]/95 backdrop-blur-xl">
              <div class="w-12 h-12 border-2 border-indigo-500 border-t-transparent rounded-full animate-spin"></div>
              <p class="mt-6 text-[10px] font-black text-indigo-400 uppercase tracking-[0.5em] animate-pulse">Running Integrated Reasoning</p>
            </div>
            
            <img v-else-if="processedImage" :src="processedImage" class="max-h-full max-w-full object-contain" />
            <div v-else class="text-slate-800 text-xs italic tracking-widest uppercase">Awaiting Process</div>
          </div>
        </div>
      </div>
    </main>

    <footer v-if="originalImage" class="w-full bg-[#161926] border-t border-white/5 p-8 flex justify-center items-center gap-8">
      <button v-if="!processedImage" @click="colorizeWithRiverflow" :disabled="isProcessing || !apiKey" class="group relative px-20 py-4 bg-indigo-600 hover:bg-indigo-500 disabled:opacity-20 rounded-full text-[11px] font-black uppercase tracking-[0.2em] text-white transition-all shadow-xl shadow-indigo-600/20 overflow-hidden">
        <span class="relative z-10">{{ isProcessing ? 'Processing...' : 'Generate Color' }}</span>
      </button>

      <template v-else>
        <button @click="downloadImage" class="px-20 py-4 bg-white text-black hover:bg-slate-200 rounded-full text-[11px] font-black uppercase tracking-[0.2em] transition-all shadow-xl">
          Download HD Render
        </button>
        <button @click="reset" class="text-indigo-400 hover:text-white text-[10px] font-bold uppercase tracking-widest">New Project</button>
      </template>
    </footer>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const apiKey = ref('');
const originalImage = ref(null);
const processedImage = ref(null);
const isProcessing = ref(false);
const base64Data = ref(null);

function handleFileUpload(event) {
  const file = event.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = (e) => {
    originalImage.value = e.target.result;
    // We send the full data URL to Riverflow V2 Pro for best compatibility
  };
  reader.readAsDataURL(file);
}

async function colorizeWithRiverflow() {
  if (!apiKey.value || !originalImage.value) return;
  isProcessing.value = true;
  processedImage.value = null;

  try {
    const response = await fetch("https://openrouter.ai/api/v1/chat/completions", {
      method: "POST",
      headers: {
        "Authorization": `Bearer ${apiKey.value}`,
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        "model": "sourceful/riverflow-v2-pro", 
        "messages": [
          {
            "role": "user",
            "content": [
              { "type": "text", "text": "Analyze this monochromatic photo and colorize it with absolute realism. Ensure the lighting and textures are natural. Output only the colorized image." },
              { "type": "image_url", "image_url": { "url": originalImage.value } }
            ]
          }
        ],
        "modalities": ["image"] // CRITICAL for 2026 image-output models
      })
    });

    const data = await response.json();
    
    // Extracting the image from the message.images array
    const message = data.choices?.[0]?.message;
    if (message && message.images && message.images.length > 0) {
      processedImage.value = message.images[0].image_url.url;
    } else {
      console.error("No image in response:", data);
      alert("No image returned. Check if your OpenRouter account has enough credits ($0.15/run).");
    }
  } catch (error) {
    console.error("Riverflow failure:", error);
    alert("API connection failed. Please check your console.");
  } finally {
    isProcessing.value = false;
  }
}

function reset() {
  originalImage.value = null;
  processedImage.value = null;
}

function downloadImage() {
  const link = document.createElement("a");
  link.href = processedImage.value;
  link.download = `riverflow-render-${Date.now()}.png`;
  link.click();
}
</script>

<style>
/* Smooth Desktop Transitions */
.animate-in { animation: fadeIn 0.8s cubic-bezier(0.16, 1, 0.3, 1) forwards; }
@keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
</style>