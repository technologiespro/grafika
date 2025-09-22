<template>
  <div>
    <div class="controls">
      <button @click="zoomIn">+</button>
      <button @click="zoomOut">-</button>
      <button @click="startCrop" :disabled="isCropping">Обрезать</button>
      <button @click="confirmCrop" v-if="isCropping">Подтвердить обрезку</button>
      <button @click="saveImage('png')">Сохранить в PNG</button>
      <button @click="saveImage('jpeg')">Сохранить в JPG</button>
    </div>
    <canvas ref="canvasEl"></canvas>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import * as fabric from 'fabric';

const canvasEl = ref<HTMLCanvasElement | null>(null);
let canvas: fabric.Canvas;
const isCropping = ref(false);
let cropRect: fabric.Rect | null = null;
let startX: number, startY: number;


const zoomIn = () => {
  const zoom = canvas.getZoom();
  canvas.setZoom(zoom * 1.1);
};

const zoomOut = () => {
  const zoom = canvas.getZoom();
  canvas.setZoom(zoom / 1.1);
};

const handleMouseWheel = (opt: fabric.IEvent<WheelEvent>) => {
  const delta = opt.e.deltaY;
  let zoom = canvas.getZoom();
  zoom *= 0.999 ** delta;
  if (zoom > 20) zoom = 20;
  if (zoom < 0.01) zoom = 0.01;
  canvas.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);
  opt.e.preventDefault();
  opt.e.stopPropagation();
};

const handlePaste = (e: ClipboardEvent) => {
  if (e.clipboardData) {
    const items = e.clipboardData.items;
    for (let i = 0; i < items.length; i++) {
      if (items[i].type.indexOf('image') !== -1) {
        const blob = items[i].getAsFile();
        if (blob) {
          const reader = new FileReader();
          reader.onload = (event) => {
            const imgElement = new Image();
            imgElement.onload = () => {
              const fabricImage = new fabric.Image(imgElement);
              canvas.add(fabricImage);
              canvas.renderAll();
            };
            imgElement.src = event.target?.result as string;
          };
          reader.readAsDataURL(blob);
        }
      }
    }
  }
};

const startCrop = () => {
  isCropping.value = true;
  canvas.selection = false; // Disable object selection
  canvas.on('mouse:down', onMouseDown);
  canvas.on('mouse:move', onMouseMove);
  canvas.on('mouse:up', onMouseUp);
};

const onMouseDown = (o: fabric.IEvent) => {
    const pointer = canvas.getPointer(o.e);
    startX = pointer.x;
    startY = pointer.y;

    cropRect = new fabric.Rect({
        left: startX,
        top: startY,
        width: 0,
        height: 0,
        fill: 'rgba(0,0,0,0.3)',
        stroke: '#ccc',
        strokeDashArray: [4, 4]
    });
    canvas.add(cropRect);
};

const onMouseMove = (o: fabric.IEvent) => {
    if (!cropRect) return;
    const pointer = canvas.getPointer(o.e);
    let width = pointer.x - startX;
    let height = pointer.y - startY;

    // Handle dragging in all directions
    if (width < 0) {
        cropRect.set({ left: pointer.x });
    }
    if (height < 0) {
        cropRect.set({ top: pointer.y });
    }

    cropRect.set({ width: Math.abs(width), height: Math.abs(height) });
    canvas.renderAll();
};

const onMouseUp = () => {
    canvas.off('mouse:down', onMouseDown);
    canvas.off('mouse:move', onMouseMove);
    canvas.off('mouse:up', onMouseUp);
};

const confirmCrop = () => {
  if (cropRect) {
    const cropped = new Image();
    cropped.src = canvas.toDataURL({
      left: cropRect.left,
      top: cropRect.top,
      width: cropRect.width,
      height: cropRect.height
    });

    cropped.onload = () => {
      canvas.clear();
      const newImage = new fabric.Image(cropped);
      canvas.add(newImage);
      canvas.renderAll();
    };
  }
  isCropping.value = false;
  cropRect = null;
  canvas.selection = true; // Re-enable object selection
};

const saveImage = (format: 'png' | 'jpeg') => {
  const dataURL = canvas.toDataURL({
    format: format,
    quality: 1
  });

  const link = document.createElement('a');
  link.href = dataURL;
  link.download = `canvas.${format}`;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
};

onMounted(() => {
  if (canvasEl.value) {
    canvas = new fabric.Canvas(canvasEl.value, {
      width: 800,
      height: 600,
      backgroundColor: '#f0f0f0'
    });
    window.addEventListener('paste', handlePaste);
    canvas.on('mouse:wheel', handleMouseWheel);
  }
});

onUnmounted(() => {
  window.removeEventListener('paste', handlePaste);
  if (canvas) {
    canvas.off('mouse:wheel', handleMouseWheel);
  }
});
</script>

<style scoped>
.controls {
  margin-bottom: 10px;
}
canvas {
  border: 1px solid #ccc;
}
</style>