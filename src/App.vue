<script setup lang="ts">
import { ref } from "vue";


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

const file = ref<File | null>(null);
const ascii = ref<string>("");
const asciiWidth = ref<number>(100);
const canvas = document.createElement("canvas");
const ctx = canvas.getContext("2d");

const img = new Image();

const previewPre = ref<HTMLPreElement | null>(null);
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
  return asciiArray[asciiIndex];
}

function UpdateWidth(width: number) {
  asciiWidth.value = width;
  RenderToASCII();
}

function ToClipboard() {
  navigator.clipboard.writeText(ascii.value);
}

function UpdatePreviewFontSize(size: number) {
  if (previewPre.value) {
    previewPre.value.style.fontSize = `${size}px`;
  }
}

</script>

<template>
  <div class="h-screen flex flex-col items-center justify-start p-8">
    <div class="bg-zinc-800 p-4 rounded-lg mb-2">

      <h1
        class="font-extrabold text-transparent text-4xl bg-clip-text bg-gradient-to-r from-purple-400 to-pink-600"
      >
      
        Image to ASCII
      </h1>
    </div>
    <div
      class="flex flex-col items-center space-y-4 bg-zinc-800 p-4 rounded-lg"
    >
      <input type="file" @change="onFileChange" class="p-2 rounded-lg" />
      <input
        type="range"
        min="25"
        max="200"
        v-model="asciiWidth"
        @change="UpdateWidth(asciiWidth)"
        class="w-64"
      />
    </div>
    <div
      class="flex flex-row w-full justify-center items-center space-x-4 mt-4"
    >
      <div
        class="w-1/2 bg-zinc-800 p-4 rounded-lg h-[700px] flex items-center justify-center"
      >
        <img :src="img.src" alt="Image" class="rounded-lg w-auto h-full" />
      </div>
      <div
        class="w-1/2 bg-zinc-800 p-4 rounded-lg overflow-auto h-[700px] relative flex flex-col justify-between"
      >
        <div class="rounded-lg overflow-auto h-[700px">
          <pre class="text-white" ref="previewPre"
          >{{ ascii }}</pre>
        </div>
        <div class="flex flex-row justify-end">
          <input type="number" v-model="previewFontSize" @change="UpdatePreviewFontSize(previewFontSize)" class="p-2 rounded-lg" />
          <button class="bg-zinc-600 p-2 rounded-lg text-white" @click="ToClipboard">
          </button>  
        </div>
      </div>
    </div>
  </div>
</template>
