<script setup lang="ts">
import { ref } from "vue";
import { onUnmounted } from "vue";
import { createApp } from "vue";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { faCopy } from "@fortawesome/free-solid-svg-icons";
import Toasts from "./components/Toasts.vue";
import ASCIIViewer from "./components/ASCIIViewer.vue";
import ImageInput from "./components/ImageInput.vue";

const asciiArray = [
  " ",
  ".",
  ",",
  ":",
  ";",
  "i",
  "1",
  "t",
  "f",
  "L",
  "C",
  "G",
  "0",
  "8",
  "#",
  "@",
];


let toastApp: ReturnType<typeof createApp> | null = null;

function ToClipboard() {
  navigator.clipboard.writeText(ascii.value);
  const toastContainer = document.createElement("div");
  document.body.appendChild(toastContainer);
  toastApp = createApp(Toasts, {
    toast: { message: "ASCII copied to clipboard!", type: "success" },
  });
  toastApp.mount(toastContainer);
}

onUnmounted(() => {
  if (toastApp) {
    toastApp.unmount();
  }
});

const file = ref<File | null>(null);
const ascii = ref<string>("");
const asciiWidth = ref<number>(100);
const canvas = document.createElement("canvas");
const ctx = canvas.getContext("2d");
const negative = ref<boolean>(false);

const img = new Image();

const previewFontSize = ref<number>(16);

const onFileChange = (e: Event) => {
  const target = e.target as HTMLInputElement;
  const files = target.files;
  if (files) {
    file.value = files[0];
    const reader = new FileReader();
    reader.onload = (e) => {
      img.src = e.target?.result as string;
      img.onload = () => {
        RenderToASCII();
      };
    };
    reader.readAsDataURL(files[0]);
  }
};

function RenderToASCII() {
  if (!ctx || !canvas || !img.src) {
    return;
  }
  const aspectRatio = img.width / img.height;
  const width = asciiWidth.value;
  const height = width / aspectRatio;
  canvas.width = width;
  canvas.height = height;
  ctx.drawImage(img, 0, 0, width, height);
  const imageData = ctx.getImageData(0, 0, width, height);
  let asciiString = "";
  for (let i = 0; i < imageData.data.length; i += 4) {
    if ((i / 4) % width === 0) {
      asciiString += "\n";
    }
    const r = imageData.data[i];
    const g = imageData.data[i + 1];
    const b = imageData.data[i + 2];
    asciiString += RGBtoASCII(r, g, b);
  }
  ascii.value = asciiString;
}

function RGBtoASCII(r: number, g: number, b: number): string {
  const gray = 0.2126 * r + 0.7152 * g + 0.0722 * b;
  const asciiIndex = Math.floor((gray * asciiArray.length) / 255);
  return negative.value
    ? asciiArray[asciiArray.length - asciiIndex - 1]
    : asciiArray[asciiIndex];
}

function UpdateWidth(width: number) {
  asciiWidth.value = width;
  RenderToASCII();
}


</script>

<template>
  <div class="h-screen flex flex-col items-center justify-start p-8">
    <div class="bg-zinc-800 p-4 rounded-lg mb-2 flex flex-col items-center">
      <h1
        class="font-extrabold text-transparent text-4xl bg-clip-text bg-gradient-to-r from-purple-400 to-pink-600"
      >
        ASCIIfier
      </h1>
      <p class="text-white text-center opacity-60">
        Convert your images to ASCII art with this simple tool.
      </p>
    </div>
    <div
      class="flex flex-col items-center space-y-4 bg-zinc-800 md:w-max w-full p-4 rounded-lg"
      v-if="!ascii"
    >
      <ImageInput :onFileChange="onFileChange" />
    </div>

    <div v-if="ascii !== ''" class="w-full">
      <div
        class="flex flex-col md:flex-row w-full justify-start items-start gap-4 mt-4 relative"
      >
        <div
          class="w-full md:w-max bg-zinc-800 p-4 rounded-lg flex items-center justify-center flex-col md:absolute md:top-4 md:left-4 z-10"
        >
          <img :src="img.src" alt="Image" class="rounded-lg w-auto h-64 min-w-64" />
          <div class="mt-4 flex flex-row items-center space-x-4">
            <label
              for="change-file"
              class="bg-zinc-600 p-2 rounded-lg text-white hover:bg-zinc-500 cursor-pointer"
            >
              Change Image
              <input
                id="change-file"
                type="file"
                @change="onFileChange"
                accept="image/svg+xml,image/png,image/jpeg"
                class="hidden"
              />
            </label>
          </div>
        </div>
        <ASCIIViewer :asciiPreview="ascii" :fontsize="previewFontSize" />
      </div>
      <div
        class="flex flex-col items-center justify-center md:absolute bottom-4 right-4 p-4 bg-zinc-800 rounded-lg mt-2 md:mt-0"
      >
        <div class="grid grid-cols-3 gap-4 w-full items-center">
          <h2 class="font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-purple-400 to-pink-600 text-center col-start-2 w-full">
            Settings</h2>
          <button
            class="bg-zinc-600 p-2 group rounded-lg col-start-3 text-white w-max gap-2 h-9 flex items-center justify-center justify-self-end"
            @click="ToClipboard"
          >
            <font-awesome-icon :icon="faCopy" />
          </button>
        </div>
        <hr class="w-full border-zinc-600 my-2" />
        <div class="flex flex-row items-center space-x-4 mt-4 w-full">
          <label for="width" class="text-white text-center w-1/2">
            Width: {{ asciiWidth }}
          </label>
          <input
            type="range"
            id="width"
            min="50"
            max="200"
            step="1"
            v-model="asciiWidth"
            @input="UpdateWidth(asciiWidth)"
            class="w-1/2"
          />
        </div>
        <div class="flex flex-row items-center space-x-4 mt-4 w-full">
          <label for="fontSize" class="text-white text-center w-1/2"
            >Font Size: {{ previewFontSize }} px</label
          >
          <input
            type="range"
            id="fontSize"
            min="8"
            max="32"
            step="1"
            v-model="previewFontSize"
            @input="RenderToASCII"
            class="w-1/2"
          />
        </div>
        <div class="flex flex-row items-center space-x-4 mt-4 w-full">
          <label for="negative" class="text-white text-center w-1/2"
            >Negative</label
          >
          <input
            type="checkbox"
            id="negative"
            v-model="negative"
            class="w-1/2"
            @change="RenderToASCII"
          />
        </div>  
      </div>
    </div>
  </div>
</template>
