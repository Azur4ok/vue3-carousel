<script setup lang="ts">
import TheCarousel from './components/TheCarousel.vue'
import { ref, onBeforeMount } from 'vue'

const images = ref([])

async function fetchImages() {
  try {
    const response = await fetch('https://picsum.photos/v2/list?limit=10')
    const data = await response.json()
    return data
  } catch (error: unknown) {
    console.log(error);
    
  }
}

onBeforeMount(async () => {
  const imgs = await fetchImages()
  images.value = imgs

 })
</script>

<template>
  <main class="w-[100vw] h-[100vh] bg-[#eff0f0] flex flex-col items-center justify-center">
    <the-carousel v-if="images.length > 0" :images="images" />
  </main>
</template>

