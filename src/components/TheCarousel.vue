<template>
    <div class="flex flex-col items-center justify-center h-full">
        <div v-if="loading" class="text-black text-3xl">Loading...</div>
        <div class="carousel-window overflow-hidden w-full max-w-screen-lg">
            <div class="carousel-track flex">
                <img v-for="(image, index) in imgs" :key="image.id" :src="image.download_url"
                    class="transition-border transition-duration-500 ease"
                    :class="`cursor-pointer ${selectedImages.includes(image.download_url) ? 'border-2 border-blue-500' : ''}`"
                    @click="toggleSelect(image.download_url)" :style="{
                        transform: `translateX(-${currentIndex * imageWidth}px)`, width: imageWidth +
                            'px'
                    }" />
            </div>
        </div>
        <div class="flex mt-4 gap-4">
            <button @click="prevImage" class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">Prev</button>
            <button @click="nextImage" class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 ">Next</button>
        </div>
        <div class="mt-4 overflow-y-scroll w-[30vw] h-[20vh] overflow-x-hidden">
            <h3 class="font-bold text-xl">Selected Images:</h3>
            <transition-group name="list" tag="ul">
                <div v-for="url in selectedImages" :key="url" class="text-sm">{{ url }}</div>
            </transition-group>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';

const props = defineProps({
    images: Array
});

const imgs = ref([]);
const currentIndex = ref(0);
const selectedImages = ref<string[]>([]);
const imageWidth = ref(400);
const imagesOnScreen = ref(3);
const loading = ref(true);

const updateDimensions = () => {
    const windowWidth = window.innerWidth;

    if (windowWidth < 640) {
        imagesOnScreen.value = 1;
    } else if (windowWidth < 1024) {
        imagesOnScreen.value = 3;
    } else {
        imagesOnScreen.value = 5;
    }

    imageWidth.value = document.querySelector('.carousel-window')!.clientWidth / imagesOnScreen.value;
};

async function preloadImages() {
    const imagePromises = props.images.map((image: any) => {
        return new Promise((resolve) => {
            const img = new Image();
            img.src = image.download_url;
            img.onload = resolve;
        });
    });

    await Promise.all(imagePromises);
    imgs.value = [...props.images.slice(-imagesOnScreen.value), ...props.images, ...props.images.slice(0, imagesOnScreen.value)];
    currentIndex.value = imagesOnScreen.value;
    loading.value = false;
};


onMounted(async () => {
    updateDimensions();
    await preloadImages();

    window.addEventListener('resize', updateDimensions);
});

onUnmounted(() => {
    window.removeEventListener('resize', updateDimensions);
});

function prevImage() {
    currentIndex.value = (currentIndex.value - 1 + imgs.value.length) % imgs.value.length;
    if (currentIndex.value === imagesOnScreen.value - 1) {
        currentIndex.value = imgs.value.length - imagesOnScreen.value;
    }
}

function nextImage() {
    currentIndex.value = (currentIndex.value + 1) % imgs.value.length;
    if (currentIndex.value === imgs.value.length - imagesOnScreen.value) {
        currentIndex.value = imagesOnScreen.value;
    }
}

const toggleSelect = (url: string) => {
    if (selectedImages.value.includes(url)) {
        selectedImages.value = selectedImages.value.filter(imgUrl => imgUrl !== url);
    } else {
        selectedImages.value.push(url);
    }
};
</script>

<style scoped>
.carousel-window {}

.list-enter-active,
.list-leave-active {
    transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
    opacity: 0;
    transform: translateX(10px);
}
</style>
