<script setup lang="ts">
import { Color } from "pixi.js";
import img from "./woman.jpg";
import { ref, reactive } from "vue"

interface Point {
	x: number;
	y: number;
	color: Color;
	radius: number
}

const GAP = 4;
const imgElement = ref()
const canvasElement = ref()

const canvasWidth = ref(800);
const canvasHeight = ref(500)

let points = reactive<Point[]>([])

function handleImageLoaded() {
	const data = getImageData(imgElement.value)
	points = getRenderPoint(data, GAP);
	drawPoints.bind(canvasElement.value)(points)
}

function getImageData(image: HTMLImageElement) {
	const canvas = document.createElement("canvas");
	canvas.width = image.width;
	canvas.height = image.height;
	const ctx = canvas.getContext("2d")
	ctx!.drawImage(image, 0, 0);
	// 获取图像数据
	const imageData = ctx!.getImageData(0, 0, canvas.width, canvas.height);
	return imageData
}

function getRenderPoint(data: ImageData, gap: number): Point[] {
	const _points: Point[] = [];
	const { width, height } = data;

	for (let y = 0; y < height; y += gap) {
		for (let x = 0; x < width; x += gap) {
			const index = (y * width + x) * 4; // 每个像素占4个值 (RGBA)

			const r = data.data[index];
			const g = data.data[index + 1];
			const b = data.data[index + 2];
			const a = data.data[index + 3];

			// 将 RGB 转换为 CSS 格式
			// const color = `rgba(${r}, ${g}, ${b}, ${a})`;
			const color = new Color({ r, g, b, a })
			const radius = getGrayScaledRadius(color)

			// 将点添加到数组
			_points.push({ x, y, color, radius });
		}
	}

	return _points;
}

function drawPoint(this: CanvasRenderingContext2D, point: Point) {
	this.fillStyle = point.color as any;
	this.beginPath();
	this.arc(point.x, point.y, point.radius, 0, Math.PI * 2);
	this.fill();
}

function getGrayScaledRadius(color: Color) {
	const grayScale = gray.call(color)
	const radius = GAP / 2 * (1 - grayScale);
	return radius
}

function gray(this: Color) {
	return 0.299 * this.red + 0.587 * this.green + 0.114 * this.blue
}

function drawPoints(this: HTMLCanvasElement, points: Point[]) {
	const ctx = this.getContext("2d");
	if (!ctx) return;

	// 清空 canvas
	ctx.clearRect(0, 0, this.width, this.height);

	// 绘制每个点
	for (let index = 0; index < points.length; index++) {
		const point = points[index];
		drawPoint.bind(ctx)(point);
	}
}


</script>

<template>
	<canvas class="canvas" ref="canvasElement" :width="canvasWidth" :height="canvasHeight"></canvas>
	<img :src="img" @load="handleImageLoaded" ref="imgElement" hidden />
</template>