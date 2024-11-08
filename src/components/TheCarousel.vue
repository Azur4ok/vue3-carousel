<template>
    <div class="flex flex-col items-center justify-center h-[98%] w-[98%]">
        <div class="relative w-full">
            <div ref="carouselWindow"
                class="carousel-container relative overflow-hidden w-full max-w-screen-lg mx-auto">
                <div class="carousel-track flex transition-transform duration-300 ease-in-out" :style="trackStyle"
                    @transitionend="handleTransitionEnd">
                    <div v-for="(image, index) in displayImages" :key="`${image.id}-${index}`"
                        class="carousel-item flex-shrink-0" :style="{ width: `${imageWidth}px` }">
                        <div class="p-2">
                            <img :src="image.download_url" :alt="`Carousel image ${index}`"
                                class="w-full h-64 object-cover rounded-lg transition-all duration-300 cursor-pointer"
                                :class="[
                                    selectedImages.includes(image.download_url) ?
                                        'ring-4 ring-blue-500 ring-offset-2' :
                                        'hover:ring-2 hover:ring-blue-300 hover:ring-offset-1'
                                ]" @click="toggleSelect(image.download_url)" />
                        </div>
                    </div>
                </div>
            </div>
            <button @click="prevImage"
                class="absolute left-0 top-1/2 -translate-y-1/2 -translate-x-2 bg-white/80 hover:bg-white p-2 rounded-full shadow-lg transition-all duration-200 group">
                <svg xmlns="http://www.w3.org/2000/svg"
                    class="h-6 w-6 text-blue-500 group-hover:scale-110 transition-transform" fill="none"
                    viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
                </svg>
            </button>
            <button @click="nextImage"
                class="absolute right-0 top-1/2 -translate-y-1/2 translate-x-2 bg-white/80 hover:bg-white p-2 rounded-full shadow-lg transition-all duration-200 group">
                <svg xmlns="http://www.w3.org/2000/svg"
                    class="h-6 w-6 text-blue-500 group-hover:scale-110 transition-transform" fill="none"
                    viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                </svg>
            </button>
            <div class="flex justify-center gap-2 mt-4">
                <button v-for="(_, index) in actualImagesLength" :key="index" @click="goToSlide(index)"
                    class="w-2 h-2 rounded-full transition-all duration-200" :class="[
                        currentActualIndex === index ?
                            'bg-blue-500 w-4' :
                            'bg-blue-200 hover:bg-blue-300'
                    ]">
                </button>
            </div>
        </div>
        <div class="mt-6 w-[30vw] h-[35vh] max-w-2xl bg-gray-50 rounded-lg p-4">
            <h3 class="font-bold text-lg text-gray-700 mb-2">Selected Images ({{ selectedImages.length }})</h3>
            <transition-group name="list" tag="ul" class="space-y-2 h-[85%] overflow-y-scroll">
                <li v-for="url in selectedImages" :key="url"
                    class="flex items-center justify-between bg-white p-2 rounded shadow-sm">
                    <span class="text-sm text-gray-600 truncate flex-1">{{ url }}</span>
                    <button @click="toggleSelect(url)" class="ml-2 text-red-500 hover:text-red-600 p-1">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24"
                            stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M6 18L18 6M6 6l12 12" />
                        </svg>
                    </button>
                </li>
            </transition-group>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, watch } from 'vue';

interface CarouselImage {
    id: string;
    download_url: string;
}

const props = defineProps<{
    images: CarouselImage[];
}>();

const carouselWindow = ref<HTMLElement | null>(null);
const currentIndex = ref(0);
const selectedImages = ref<string[]>([]);
const imageWidth = ref(0);
const imagesPerView = ref(5);
const isTransitioning = ref(false);
const totalClones = ref(4);

const displayImages = computed(() => {
    const cloneCount = totalClones.value;
    const images = [...props.images];
    const result = [];

    for (let i = 0; i < cloneCount; i++) {
        result.push(...images);
    }

    result.push(...images);

    for (let i = 0; i < cloneCount; i++) {
        result.push(...images);
    }

    return result;
});

const actualImagesLength = computed(() => props.images.length);

const currentActualIndex = computed(() => {
    return Math.floor(currentIndex.value % actualImagesLength.value);
});

const trackStyle = computed(() => {
    const baseTransform = currentIndex.value * -imageWidth.value;
    const offset = totalClones.value * actualImagesLength.value * imageWidth.value;

    return {
        transform: `translateX(${baseTransform + offset}px)`,
        transition: isTransitioning.value ? 'transform 300ms ease-in-out' : 'none'
    };
});

const updateDimensions = () => {
    if (!carouselWindow.value) return;

    const containerWidth = carouselWindow.value.clientWidth;

    if (containerWidth < 640) {
        imagesPerView.value = 1;
    } else if (containerWidth < 1024) {
        imagesPerView.value = 3;
    } else {
        imagesPerView.value = 5;
    }

    imageWidth.value = containerWidth / imagesPerView.value;
};

const goToSlide = (index: number) => {
    isTransitioning.value = true;
    currentIndex.value = index + totalClones.value * actualImagesLength.value;
};

const handleTransitionEnd = () => {
    isTransitioning.value = false;

    const totalImages = actualImagesLength.value;
    const currentPosition = currentIndex.value % totalImages;
    const targetIndex = totalClones.value * totalImages + currentPosition;

    if (currentIndex.value !== targetIndex) {
        currentIndex.value = targetIndex;
    }
};

const nextImage = () => {
    isTransitioning.value = true;
    currentIndex.value++;
};

const prevImage = () => {
    isTransitioning.value = true;
    currentIndex.value--;
};

const toggleSelect = (url: string) => {
    const index = selectedImages.value.indexOf(url);
    if (index === -1) {
        selectedImages.value.push(url);
    } else {
        selectedImages.value.splice(index, 1);
    }
};

onMounted(async () => {
    updateDimensions();

    currentIndex.value = totalClones.value * actualImagesLength.value;

    window.addEventListener('resize', updateDimensions);
});

onUnmounted(() => {
    window.removeEventListener('resize', updateDimensions);
});

watch([imagesPerView], updateDimensions);
</script>

<style scoped>
.carousel-track {
    will-change: transform;
}

.list-enter-active,
.list-leave-active {
    transition: all 0.3s ease;
}

.list-enter-from,
.list-leave-to {
    opacity: 0;
}

.list-move {
    transition: transform 0.3s ease;
}
</style>