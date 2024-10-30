<script setup lang="ts">
import { ref, onMounted, reactive } from "vue";
import { Application, Assets, Sprite, ColorMatrixFilter } from 'pixi.js';
import img from "./zhang.png"
const pixiContainerRef = ref();
let app: Application;

async function loadImage() {
  const result = await Assets.load(img)
  return result
}

const GRAY = 0.5;

async function draw() {
  const texture = await loadImage();
  const sprite = new Sprite(texture);
  const colorMatrix = new ColorMatrixFilter();
  colorMatrix.grayscale(GRAY, true);
  sprite.filters = [colorMatrix];
  app.stage.addChild(sprite);
}

onMounted(async () => {
  app = new Application();
  //创建了一个800*600 的canvas
  await app.init({
    background: "white"
  });
  if (pixiContainerRef.value) {
    pixiContainerRef.value!.appendChild(app.canvas);
  };
  draw();
})

</script>
<template>
  <div class="pixi-container" ref="pixiContainerRef" />
  <img :src="img">
</template>
