<script setup lang="ts">
import { defineProps, onUnmounted } from "vue";
import { createApp } from 'vue';
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { faCopy } from "@fortawesome/free-solid-svg-icons";
import Toasts from "./Toasts.vue";

const props = defineProps<{ asciiPreview: string }>();
let toastApp : ReturnType<typeof createApp> | null = null;

function ToClipboard() {
  navigator.clipboard.writeText(props.asciiPreview);
  const toastContainer = document.createElement('div');
  document.body.appendChild(toastContainer);
  toastApp = createApp(Toasts, { toast: { message: 'Texte copié dans le presse-papiers', type: 'success' } });
  toastApp.mount(toastContainer);
}

onUnmounted(() => {
  if (toastApp) {
    toastApp.unmount();
  }
});
</script>

<template>
  <div
    class="w-1/2 bg-zinc-800 p-4 rounded-lg overflow-auto h-[700px] relative flex flex-col justify-between"
  >
    <div class="rounded-lg overflow-auto h-[700px">
      <pre class="text-white" ref="previewPre">{{ props.asciiPreview }}</pre>
    </div>
    <div class="flex flex-row justify-end">
      <button
        class="bg-zinc-600 p-2 rounded-lg text-white"
        @click="ToClipboard"
      >
        <font-awesome-icon :icon="faCopy" />
      </button>
    </div>
  </div>
</template>
