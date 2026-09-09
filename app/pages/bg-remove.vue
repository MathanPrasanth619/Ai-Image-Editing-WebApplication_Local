<template>
  <div class="max-w-5xl mx-auto p-6 space-y-8">
    <div class="flex items-center justify-between">
      <NuxtLink to="/" class="btn btn-ghost btn-sm gap-2 text-white">
        <span>←</span> Back
      </NuxtLink>
      <h2 class="text-xl font-bold uppercase tracking-widest text-white">
        BG Remover
      </h2>
    </div>

    <div
      class="bg-slate-900 shadow-xl border border-white/10 overflow-hidden rounded-4xl"
    >
      <div class="p-4 flex flex-col items-center gap-6">
        <label
          v-if="!originalImage"
          class="group flex flex-col items-center justify-center w-112.5 h-64 border-2 border-dashed border-white/20 rounded-3xl cursor-pointer hover:bg-slate-800 transition-all"
        >
          <div
            class="flex flex-col items-center justify-center pt-5 pb-6 text-center"
          >
            <p class="mb-2 text-sm text-white/60">
              <span class="font-semibold text-white">Click to upload</span> or
              drag and drop
            </p>
            <p class="text-xs text-white/40">PNG, JPG or WEBP</p>
          </div>
          <input
            type="file"
            ref="fileInput"
            class="hidden"
            @change="handleFileUpload"
            accept="image/*"
          />
        </label>

        <div
          v-if="originalImage"
          class="w-full flex flex-col items-center gap-6"
        >
          <div
            class="relative w-full max-w-2xl aspect-video bg-black rounded-2xl overflow-hidden shadow-inner flex items-center justify-center border border-white/5"
          >
            <figure
              v-if="processedImage"
              class="diff aspect-video w-full"
              tabindex="0"
            >
              <div class="diff-item-1 size-full" role="img" tabindex="0">
                <img :src="originalImage" alt="Original" />
              </div>
              <div class="diff-item-2 size-full" role="img">
                <img :src="processedImage" alt="Background Removed" />
              </div>
              <div class="diff-resizer"></div>
            </figure>

            <div
              v-else-if="isProcessing"
              class="flex flex-col items-center gap-4 w-112.5"
            >
              <span
                class="loading loading-infinity loading-lg text-primary"
              ></span>
              <p class="animate-pulse font-medium text-white">
                Processing AI Model...
              </p>
            </div>

            <img
              v-else
              :src="originalImage"
              alt="Preview"
              class="max-h-full object-contain p-4"
            />
          </div>

          <div class="flex gap-4 mb-2">
            <button
              v-if="!processedImage"
              @click="removeBg"
              :disabled="isProcessing"
              class="btn btn-primary px-10 rounded-full shadow-lg"
            >
              {{ isProcessing ? "Removing..." : "Remove Background" }}
            </button>

            <button
              v-if="processedImage"
              @click="downloadImage"
              class="btn btn-secondary px-10 rounded-full shadow-lg"
            >
              Download PNG
            </button>

            <button
              @click="reset"
              class="btn btn-error px-10 rounded-full text-white"
            >
              Start Over
            </button>
          </div>
        </div>

        <div v-if="error" class="text-error text-sm font-medium">
          {{ error }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import * as BackgroundRemoval from "@imgly/background-removal";

const fileInput = ref(null);
const originalImage = ref(null);
const processedImage = ref(null);
const isProcessing = ref(false);
const error = ref(null);

function handleFileUpload(event) {
  const file = event.target.files[0];
  if (!file) return;

  error.value = null;
  processedImage.value = null;

  const reader = new FileReader();
  reader.onload = (e) => {
    originalImage.value = e.target.result;
  };
  reader.readAsDataURL(file);
}

async function removeBg() {
  isProcessing.value = true;
  error.value = null;

  try {
    const result = await BackgroundRemoval.removeBackground(
      originalImage.value,
    );
    processedImage.value = URL.createObjectURL(result);
  } catch (err) {
    error.value =
      "Failed to process image. Make sure your browser supports WebAssembly.";
    console.error(err);
  } finally {
    isProcessing.value = false;
  }
}

function downloadImage() {
  const link = document.createElement("a");
  link.href = processedImage.value;
  link.download = "no-background.png";
  link.click();
}

function reset() {
  originalImage.value = null;
  processedImage.value = null;
  error.value = null;
}
</script>

