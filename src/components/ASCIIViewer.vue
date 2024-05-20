<script setup lang="ts">
import { defineProps, ref, watch } from "vue";
const props = defineProps<{ asciiPreview: string; fontsize: number }>();

const previewPre = ref<HTMLPreElement | null>(null);

watch(
  () => props.fontsize,
  () => {
    if (previewPre.value) {
      console.log(props.fontsize);
    previewPre.value.style.fontSize = `${props.fontsize}px`;
    }
  }
);
</script>

<template>
  <div class="flex items-center justify-center w-full h-full">
    <div
      class="bg-zinc-800 p-4 rounded-lg overflow-auto relative flex flex-col justify-between"
    >
      <div class="rounded-lg overflow-auto max-h-[80vh] w-full">
          <pre class="text-white w-full" ref="previewPre">{{
          props.asciiPreview
        }}</pre>
      </div>
    </div>
  </div>
</template>
