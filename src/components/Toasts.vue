<script setup lang="ts">
import { defineProps } from "vue";

const props = defineProps<{ toast: { message: string; type: toastTypes } }>();

type toastTypes = "success" | "error" | "warning";

const toastClasses = {
  success: "bg-green-400",
  error: "bg-red-500",
  warning: "bg-yellow-500",
};

const toastClass = toastClasses[props.toast.type];

const removeToast = () => {
  const toast = document.getElementById("toast");
  if (toast) {
    setTimeout(() => {
      toast.remove();
    }, 500);
  }
};

setTimeout(() => {
  removeToast();
}, 5000);
</script>

<template>
  <div
    id="toast"
    class="fixed bottom-4 right-4 p-4 rounded-lg text-white"
    :class="toastClass"
  >
    {{ props.toast.message }}
  </div>
</template>
