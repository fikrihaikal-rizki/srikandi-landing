<script setup>
import { ref, computed, onMounted, nextTick } from "vue";

const props = defineProps({
  path: { type: String, required: true },
  alt: { type: String, default: "Gambar Properti" },
  containerClass: { type: String, default: "" },
  imageClass: { type: String, default: "" },
  priority: { type: Boolean, default: false },
});

const isLoaded = ref(false);
const imgRef = ref(null);
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

const onImageLoad = () => {
  setTimeout(() => {
    isLoaded.value = true;
  }, 50);
};

onMounted(async () => {
  await nextTick();
  if (imgRef.value?.complete && imgRef.value?.naturalWidth > 0) {
    onImageLoad();
  }
});
</script>

<style scoped>
img {
  will-change: transform, opacity, filter;
  backface-visibility: hidden;
}
</style>

<template>
  <div :class="containerClass">
    <img
      :src="thumbUrl"
      :alt="alt"
      class="absolute inset-0 w-full h-full object-cover blur-xl scale-110 transition-opacity duration-1000 ease-in-out"
      :class="isLoaded ? 'opacity-0 pointer-events-none' : 'opacity-100'"
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
</template>
