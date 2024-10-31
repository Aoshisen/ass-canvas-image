<script setup lang="ts">
import { ref, reactive, onMounted } from "vue"
import { randomUniform, color as d3Color, RGBColor } from "d3";
import { useEventListener, useThrottleFn } from "@vueuse/core";
const props = defineProps<{
	src: string;
}>()

interface Point {
	x: number;
	y: number;
	color: RGBColor;
	radius: number;
	offsetX: number;
	offsetY: number;
}

const GAP = 4;
const canvasElement = ref()

let points = reactive<Point[]>([])

function handleImageLoaded(this: HTMLImageElement) {
	const data = getImageData(this)
	points = getRenderPoint(data, GAP);
	drawPoints.bind(canvasElement.value)(points)
}

function animation(step: number) {
	const currentPoints = points.map((point) => getNextPoint(point, step))
	drawPoints.bind(canvasElement.value)(currentPoints)
}

function getImageData(image: HTMLImageElement) {
	const canvas = document.createElement("canvas");
	const W = canvas.width = image.width;
	const H = canvas.height = image.height;
	const ctx = canvas.getContext("2d")
	ctx!.drawImage(image, 0, 0);
	const imageData = ctx!.getImageData(0, 0, W, H);
	return imageData;
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
			const color = d3Color(`rgba(${r},${g},${b},${a})`) as RGBColor;
			const radius = getGrayScaledRadius(color);
			const offsetX = randomUniform(0, 6)();
			const offsetY = randomUniform(0, 6)();
			_points.push({ x, y, color, radius, offsetX, offsetY });
		}
	}

	return _points;
}

function drawPoint(this: CanvasRenderingContext2D, point: Point) {
	this.fillStyle = point.color.toString();
	this.beginPath();
	this.arc(point.x, point.y, point.radius, 0, Math.PI * 2);
	this.fill();
}

function getGrayScaledRadius(color: RGBColor) {
	const grayScale = gray.call(color) / 255;
	const radius = GAP / 2 * (1 - grayScale);
	return radius;
}

function gray(this: RGBColor) {
	return 0.299 * this.r + 0.587 * this.g + 0.114 * this.b
}

function drawPoints(this: HTMLCanvasElement, points: Point[]) {
	const ctx = this.getContext("2d");
	if (!ctx) return;
	ctx.clearRect(0, 0, this.width, this.height);
	points.forEach(point => drawPoint.bind(ctx)(point))
}

function handleMounted() {
	const image = new Image()
	image.src = props.src;
	image.onload = handleImageLoaded.bind(image);
}


function getNextPoint(point: Point, step: number) {
	return {
		...point,
		x: point.x + point.offsetX * step,
		y: point.y + point.offsetY * step
	};
}

function scroll() {
	animation(window.scrollY / 20)
}

onMounted(handleMounted)
const throttledScroll = useThrottleFn(scroll, 20)
useEventListener(window, "scroll", throttledScroll)
</script>

<template>
	<div style="height:200px"></div>
	<canvas ref="canvasElement" width="800" height="600"></canvas>
	<div style="height:1200px"></div>
</template>