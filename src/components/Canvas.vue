<script setup lang="ts">
import { ref, onMounted } from "vue";
import img from "./zhang.png"
import * as d3 from "d3";
const imageRef = ref();
let svgHeight = ref(200);
let svgWidth = ref(200);

const renderPoints = ref([{ x: 0, y: 0, color: "#000" }])

const DOT_SIZE = 2; // 最大圆点大小
const GAP = 4;     // 点之间的间距

function load() {
  const image_data = getImageData(imageRef.value)
  svgHeight.value = imageRef.value.height;
  svgWidth.value = imageRef.value.width;
  renderPoints.value = extractFeaturePoints(image_data, imageRef.value.width, imageRef.value.height, GAP)
}

function getImageData(image: HTMLImageElement) {
  const canvas = document.createElement('canvas');
  canvas.width = image.width;
  canvas.height = image.height;

  const context = canvas.getContext('2d');
  context!.drawImage(image, 0, 0);

  const imageData = context!.getImageData(0, 0, canvas.width, canvas.height);
  return imageData.data
}

function extractFeaturePoints(data: Uint8ClampedArray, imgWidth: number, imgHeight: number, gap: number) {
  const features = [];

  for (let y = 0; y < imgHeight; y += gap) {
    for (let x = 0; x < imgWidth; x += gap) {
      const pixelIndex = (y * imgWidth + x) * 4;
      const r = data[pixelIndex];
      const g = data[pixelIndex + 1];
      const b = data[pixelIndex + 2];
      const alpha = data[pixelIndex + 3]; // 获取 alpha 通道值

      // 只处理非透明的像素点
      if (alpha > 0) {
        const gray_value = Math.round(getGrayValue(r, g, b));
        const color = d3.color(`rgb(${gray_value},${gray_value},${gray_value})`);
        features.push({ x, y, color});
      }
    }
  }

  return features;
}

function getGrayValue(r: number, g: number, b: number) {
  const gray = (r * 0.2989 + g * 0.5870 + b * 0.1140);
  return gray;
}

onMounted(() => {
  imageRef.value.addEventListener("load", load)
})

</script>
<template>
  <img :src="img" ref="imageRef" hidden />
  <svg :width="svgWidth" :height="svgHeight" style="background: white;">
    <circle v-for="(point, index) in renderPoints" :key="index" :cx="point.x" :cy="point.y"
      :r="DOT_SIZE" :fill="point.color" />
  </svg>
</template>
