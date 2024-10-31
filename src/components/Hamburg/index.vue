<script setup lang="ts">
import { ref, onMounted } from "vue";
import { Application, Assets, Sprite, ColorMatrixFilter, Container, ApplicationOptions } from 'pixi.js';
import { DotFilter } from "pixi-filters";
import img from "./hamburg.png"

const pixiContainerRef = ref();

const grayFilter = new ColorMatrixFilter();
const dotFilter = new DotFilter()

const options: Partial<ApplicationOptions> = {
  backgroundColor: "white",
}

const GRAY = 0.5;

async function loadImage() {
  const result = await Assets.load(img)
  return result
}

async function draw(this: Container) {
  //添加素材
  const texture = await loadImage();
  const sprite = new Sprite(texture);

  //设置filter
  grayFilter.grayscale(GRAY, false);
  dotFilter.grayscale = false;

  //添加filter
  sprite.filters = [grayFilter, dotFilter];

  //添加素材到显示层
  this.addChild(sprite);
}

async function beforeMount() {
  const app = new Application();
  const container = new Container()
  app.stage.addChild(container)
  await app.init(options)
  return { canvas: app.canvas, container: container };
}

function mount(this: HTMLElement, canvas: HTMLCanvasElement) {
  this?.appendChild(canvas)
}

onMounted(async () => {
  const app = await beforeMount()
  mount.bind(pixiContainerRef.value)(app.canvas)
  draw.call(app.container);
})

</script>

<template>
  <div class="pixi-container" ref="pixiContainerRef" />
</template>
