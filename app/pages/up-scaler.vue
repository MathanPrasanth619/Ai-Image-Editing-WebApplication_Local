<template>
  <div class="max-w-5xl mx-auto p-6 space-y-8">
    <div class="flex items-center justify-between">
      <NuxtLink to="/" class="btn btn-ghost btn-sm gap-2">
        <span class="opacity-50">←</span> Back
      </NuxtLink>
      <div class="flex flex-col items-end">
        <h2 class="text-xl font-bold uppercase tracking-widest text-primary">
          Up-Scaler
        </h2>
        <span class="text-[10px] opacity-50 font-mono">POWERED BY ESRGAN</span>
      </div>
    </div>

    <div
      class="bg-base-200 shadow-2xl border border-base-content/5 overflow-hidden rounded-4xl"
    >
      <div class="p-4 flex flex-col items-center gap-6">
        <label
          v-if="!originalImage"
          class="group flex flex-col items-center justify-center w-full h-80 border-2 border-dashed border-primary/20 rounded-[2rem] cursor-pointer hover:border-primary/50 hover:bg-primary/5 transition-all duration-300"
        >
          <div class="flex flex-col items-center justify-center space-y-4">
            <div
              class="size-16 rounded-full bg-primary/10 flex items-center justify-center group-hover:scale-110 transition-transform"
            >
              <span class="text-3xl">↑</span>
            </div>
            <div class="text-center">
              <p class="text-lg font-medium">Drop your image to enhance</p>
              <p class="text-xs opacity-50">
                Supports high-resolution reconstruction
              </p>
            </div>
          </div>
          <input
            type="file"
            class="hidden"
            @change="upScaleImage"
            accept="image/*"
          />
        </label>

        <div v-else class="w-full flex flex-col items-center gap-8">
          <div
            class="relative group w-full max-w-3xl rounded-2xl overflow-hidden bg-neutral shadow-inner ring-1 ring-white/10"
          >
            <img
              :src="processImage || originalImage"
              class="w-full h-auto object-contain max-h-[60vh]"
              :class="{ 'blur-sm opacity-50': isProcessing }"
              alt="Upscale Preview"
            />

            <div
              v-if="isProcessing"
              class="absolute inset-0 flex flex-col items-center justify-center bg-base-300/60 backdrop-blur-md"
            >
              <span
                class="loading loading-spinner loading-lg text-primary"
              ></span>
              <p
                class="mt-4 font-mono text-xs uppercase tracking-tighter animate-pulse"
              >
                Reconstructing Pixels...
              </p>
            </div>

            <div
              v-if="processImage && !isProcessing"
              class="absolute top-4 right-4 badge badge-primary py-3 px-4 font-bold shadow-lg"
            >
              4X ENHANCED
            </div>
          </div>

          <div class="flex items-center gap-4">
            <button
              v-if="!isProcessing"
              @click="downloadImage"
              class="btn btn-primary btn-wide rounded-full shadow-xl shadow-primary/20"
            >
              Download Enhanced Image
            </button>

            <button @click="reset" class="btn btn-ghost rounded-full px-8">
              New Image
            </button>
          </div>
        </div>
      </div>
    </div>

    <div
      class="grid grid-cols-1 md:grid-cols-3 gap-6 opacity-60 text-center text-sm"
    >
      <div class="p-4">Pixel-perfect scaling</div>
      <div class="p-4 border-x border-base-content/10">AI Noise Reduction</div>
      <div class="p-4">Edge Sharpening</div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import Upscaler from "upscaler";

const originalImage = ref(null);
const processImage = ref(null);
const isProcessing = ref(false);

async function upScaleImage(event) {
  const file = event.target.files[0];
  if (!file) return;

  // Set original for preview
  originalImage.value = URL.createObjectURL(file);
  processImage.value = null;
  isProcessing.value = true;

  try {
    const upscaler = new Upscaler();
    const result = await upscaler.upscale(originalImage.value);
    processImage.value = result;
  } catch (err) {
    console.error("Upscaling failed:", err);
  } finally {
    isProcessing.value = false;
  }
}

function downloadImage() {
  const link = document.createElement("a");
  link.href = processImage.value;
  link.download = "upscaled-ai-image.png";
  link.click();
}

function reset() {
  originalImage.value = null;
  processImage.value = null;
  isProcessing.value = false;
}
</script>

