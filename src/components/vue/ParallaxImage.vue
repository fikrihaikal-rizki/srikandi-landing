<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick } from "vue";

const props = defineProps({
  path: { type: String, required: true },
  alt: { type: String, default: "Gambar Properti" },
  containerClass: { type: String, default: "" },
  imageClass: { type: String, default: "" },
  priority: { type: Boolean, default: false },
  speed: { type: Number, default: 0.15 },
});

const isLoaded = ref(false);
const imgRef = ref(null);
const containerRef = ref(null);
const yOffset = ref(0);

const endpoint = "https://ik.imagekit.io/6066gz5in/SRIKANDI/";

const mainUrl = computed(() => `${endpoint}${props.path}?tr=w-1200,q-100`);
const thumbUrl = computed(() => `${endpoint}${props.path}?tr=w-50,q-10,bl-10`);
const srcSet = computed(() => {
  const widths = [480, 768, 1024, 1280, 1600, 2000];
  return widths
    .map((w) => `${endpoint}${props.path}?tr=w-${w},q-100 ${w}w`)
    .join(", ");
});
const sizes = "(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 1200px";

// --- Logic Parallax ---
const handleParallax = () => {
  if (!containerRef.value) return;
  const rect = containerRef.value.getBoundingClientRect();
  const viewportHeight = window.innerHeight;

  if (rect.top < viewportHeight && rect.bottom > 0) {
    const distanceToCenter = rect.top - (viewportHeight / 2 - rect.height / 2);
    yOffset.value = distanceToCenter * props.speed;
  }
};

const onImageLoad = () => {
  setTimeout(() => {
    isLoaded.value = true;
  }, 50);
};

onMounted(async () => {
  window.addEventListener("scroll", handleParallax, { passive: true });
  handleParallax(); // Initial position

  await nextTick();
  if (imgRef.value?.complete && imgRef.value?.naturalWidth > 0) {
    onImageLoad();
  }
});

onUnmounted(() => {
  window.removeEventListener("scroll", handleParallax);
});
</script>

<template>
  <div
    ref="containerRef"
    class="relative overflow-hidden"
    :class="containerClass"
  >
    <div
      class="absolute -top-[10%] left-0 w-full h-[calc(100%)] will-change-transform bg-red-300"
      :style="{ transform: `translate3d(0, ${yOffset}px, 0)` }"
    >
      <img
        :src="thumbUrl"
        :alt="alt"
        class="absolute inset-0 w-full h-full object-cover blur-xl scale-110 transition-opacity duration-1000 ease-in-out"
        :class="isLoaded ? 'opacity-0' : 'opacity-100'"
      />

      <img
        ref="imgRef"
        :src="mainUrl"
        :srcset="srcSet"
        :sizes="sizes"
        :alt="alt"
        :loading="priority ? 'eager' : 'lazy'"
        :decoding="priority ? 'sync' : 'async'"
        class="transition-all duration-1000 ease-out"
        :class="[
          isLoaded ? 'opacity-100 scale-100' : 'opacity-0 scale-105',
          imageClass,
        ]"
        @load="onImageLoad"
      />
    </div>
  </div>
</template>

<style scoped>
img,
.will-change-transform {
  will-change: transform;
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
}

img {
  object-fit: cover;
}
</style>
