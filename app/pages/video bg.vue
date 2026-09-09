<template>
  <div class="max-w-5xl mx-auto p-6 space-y-8">
    <div class="flex items-center justify-between text-white">
      <NuxtLink to="/" class="btn btn-ghost btn-sm">← Back</NuxtLink>
      <h2 class="text-xl font-bold tracking-widest uppercase">AI Video BackgroundRemoval</h2>
    </div>

    <div class="bg-slate-900 shadow-2xl border border-white/10 rounded-3xl overflow-hidden p-6">
      <div class="flex flex-col items-center gap-6">
        
        <label v-if="!originalVideo" class="w-full h-64 border-2 border-dashed border-white/20 rounded-2xl flex flex-col items-center justify-center cursor-pointer hover:bg-slate-800 transition-all">
          <div class="text-center">
            <p class="text-white font-semibold">Upload Video (MP4/MOV)</p>
            <p class="text-white/40 text-xs mt-1">Recommended: 720p or lower</p>
          </div>
          <input type="file" class="hidden" @change="handleFileUpload" accept="video/*" />
        </label>

        <div v-if="originalVideo" class="w-full flex flex-col items-center gap-6">
          <div class="relative w-full max-w-2xl aspect-video bg-black rounded-xl overflow-hidden border border-white/5 shadow-inner flex items-center justify-center">
            
            <video v-if="processedVideo" :src="processedVideo" class="w-full h-full object-contain" controls autoplay loop></video>
            
            <div v-else-if="isProcessing" class="absolute inset-0 flex flex-col items-center justify-center bg-slate-900/90 backdrop-blur-md">
              <div class="text-center">
                <div class="radial-progress text-primary mb-4" :style="`--value:${processingProgress}; --size:6rem;`" role="progressbar">
                  {{ processingProgress }}%
                </div>
                <p class="text-white font-bold tracking-tight">AI REMOVING BACKGROUND...</p>
                <p class="text-white/40 text-xs mt-1 uppercase">{{ backendStatus }}</p>
              </div>
            </div>

            <video v-else :src="originalVideo" class="w-full h-full object-contain"></video>
          </div>

          <div class="flex gap-4">
            <button v-if="!processedVideo" @click="removeVideoBg" :disabled="isProcessing" class="btn btn-primary px-12 rounded-full shadow-lg shadow-primary/20">
              {{ isProcessing ? "Processing..." : "Start Processing" }}
            </button>
            <button v-if="processedVideo" @click="downloadVideo" class="btn btn-secondary px-10 rounded-full">Download WebM</button>
            <button @click="reset" class="btn btn-ghost text-white/50">Reset</button>
          </div>
        </div>

        <canvas ref="processingCanvasEl" class="hidden"></canvas>
        <video ref="sourceVideoEl" class="hidden" muted playsinline crossorigin="anonymous"></video>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import * as ort from "onnxruntime-web";

const originalVideo = ref(null);
const processedVideo = ref(null);
const isProcessing = ref(false);
const processingProgress = ref(0);
const backendStatus = ref("Initializing...");
const sourceVideoEl = ref(null);
const processingCanvasEl = ref(null);

let modnetSession = null;

async function initModel() {
  if (modnetSession) return;
  ort.env.wasm.wasmPaths = "/";
  
  try {
    // Attempt GPU (WebGL) first
    backendStatus.value = "Starting GPU Acceleration...";
    modnetSession = await ort.InferenceSession.create("/models/modnet.onnx", {
      executionProviders: ["webgl"],
    });
    backendStatus.value = "Mode: GPU (Fast)";
  } catch (e) {
    console.warn("GPU failed, switching to CPU fallback.");
    // Fallback to WASM (CPU)
    modnetSession = await ort.InferenceSession.create("/models/modnet.onnx", {
      executionProviders: ["wasm"],
    });
    backendStatus.value = "Mode: CPU (Stable)";
  }
}

function handleFileUpload(event) {
  const file = event.target.files[0];
  if (file) originalVideo.value = URL.createObjectURL(file);
}

async function removeVideoBg() {
  isProcessing.value = true;
  try {
    await initModel();
    const video = sourceVideoEl.value;
    video.src = originalVideo.value;
    await new Promise(r => video.onloadedmetadata = r);

    const canvas = processingCanvasEl.value;
    const ctx = canvas.getContext("2d", { willReadFrequently: true });
    
    // We use 384 for the AI, but we will output 720p if needed. 
    // Keeping this at 384 ensures it doesn't crash your GPU.
    const dim = 384; 
    canvas.width = dim;
    canvas.height = dim;

    // Fix the 2-minute duration bug: We use a fixed frame rate capture
    const stream = canvas.captureStream(0); // Manual frame capture
    const recorder = new MediaRecorder(stream, { 
      mimeType: "video/webm;codecs=vp9",
      videoBitsPerSecond: 6000000 
    });
    
    const chunks = [];
    recorder.ondataavailable = (e) => chunks.push(e.data);
    recorder.start();

    const fps = 24;
    const totalFrames = Math.floor(video.duration * fps);

    for (let i = 0; i < totalFrames; i++) {
      video.currentTime = i / fps;
      await new Promise(r => video.onseeked = r);

      ctx.drawImage(video, 0, 0, dim, dim);
      const imgData = ctx.getImageData(0, 0, dim, dim);
      
      const input = new Float32Array(3 * dim * dim);
      for (let j = 0; j < dim * dim; j++) {
        input[j] = (imgData.data[j * 4] / 255 - 0.5) / 0.5;
        input[dim * dim + j] = (imgData.data[j * 4 + 1] / 255 - 0.5) / 0.5;
        input[2 * dim * dim + j] = (imgData.data[j * 4 + 2] / 255 - 0.5) / 0.5;
      }

      const tensor = new ort.Tensor("float32", input, [1, 3, dim, dim]);
      const output = await modnetSession.run({ [modnetSession.inputNames[0]]: tensor });
      const mask = output[modnetSession.outputNames[0]].data;

      for (let j = 0; j < dim * dim; j++) {
        imgData.data[j * 4 + 3] = mask[j] * 255;
      }
      ctx.putImageData(imgData, 0, 0);

      // Manually push the frame to the recorder to keep duration perfect
      stream.getVideoTracks()[0].requestFrame();
      processingProgress.value = Math.round((i / totalFrames) * 100);
    }

    recorder.stop();
    processedVideo.value = URL.createObjectURL(
      await new Promise(r => recorder.onstop = () => r(new Blob(chunks, { type: "video/webm" })))
    );
  } catch (e) {
    console.error(e);
    backendStatus.value = "Error: " + e.message;
  } finally {
    isProcessing.value = false;
  }
}

function downloadVideo() {
  const a = document.createElement("a");
  a.href = processedVideo.value;
  a.download = "output.webm";
  a.click();
}

function reset() {
  originalVideo.value = null;
  processedVideo.value = null;
  processingProgress.value = 0;
}
</script>