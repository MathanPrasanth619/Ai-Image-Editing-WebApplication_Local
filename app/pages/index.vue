<template>
  <div class="min-h-[80vh] flex flex-col items-center justify-center p-6">
    <div class="text-center mb-12 space-y-4">
      <h1 class="text-6xl font-black tracking-tighter bg-gradient-to-br from-primary to-accent bg-clip-text text-transparent">
        VirtualFixer
      </h1>
      <p class="text-base-content/60 text-lg max-w-md mx-auto">
        Professional-grade AI image tools right in your browser.
      </p>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-6 w-full max-w-4xl">
      <NuxtLink
        v-for="item in navList"
        :key="item.name"
        :to="item.link"
        class="group relative overflow-hidden rounded-[2rem] bg-base-200 p-8 transition-all duration-300 hover:bg-base-300 hover:scale-[1.02] border border-base-content/5 active:scale-95 shadow-sm hover:shadow-xl"
      >
        <div class="flex flex-col h-full justify-between gap-8">
          <div class="size-14 rounded-2xl bg-primary/10 flex items-center justify-center text-primary group-hover:bg-primary group-hover:text-primary-content transition-colors duration-500">
            <span v-if="item.name === 'BG Remove'" class="text-2xl">✂️</span>
            <span v-else-if="item.name === 'Up Scaler'" class="text-2xl">💎</span>
            <span v-else class="text-2xl">✨</span>
          </div>

          <div>
            <div class="flex items-center gap-2">
              <h3 class="text-2xl font-bold tracking-tight">{{ item.name }}</h3>
              <span class="badge badge-sm badge-outline opacity-50 group-hover:opacity-100 transition-opacity">AI</span>
            </div>
            <p class="text-sm text-base-content/50 mt-2 leading-relaxed">
              {{ item.description }}
            </p>
          </div>
          
          <div class="absolute bottom-8 right-8 opacity-0 group-hover:opacity-100 group-hover:translate-x-2 transition-all">
            <span>→</span>
          </div>
        </div>
      </NuxtLink>
    </div>
  </div>
</template>

<script setup>
const navList = [
  {
    name: "BG Remove",
    link: "/bg-remove",
    description: "Instantly remove backgrounds from images with high precision using IMG.LY."
  },
  {
    name: "Up Scaler",
    link: "/up-scaler",
    description: "Transform low-resolution photos into high-definition masterpieces using ESRGAN."
  }
];
</script>