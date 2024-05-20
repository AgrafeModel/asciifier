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
    toast.classList.remove("appear");
    toast.classList.add("disappear");
    setTimeout(() => {
      toast.remove();
    }, 1000);
  }
};

setTimeout(() => {
  removeToast();
}, 2000);
</script>

<template>
  <div
    id="toast"
    class="fixed bottom-4 right-4 p-4 rounded-lg text-white appear"
    :class="toastClass"
    :on-click="removeToast"
  >
    {{ props.toast.message }}
  </div>
</template>

<style scoped>

.disappear {
  animation: disappear 1s forwards;
}

.appear {
  animation: appear 0.5s forwards;
}

@keyframes appear {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

@keyframes disappear {
  0% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}

</style>
