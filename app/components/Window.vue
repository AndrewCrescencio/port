<script setup lang="ts">
import { computed, onUnmounted, ref } from 'vue';
import { Maximize2, Minus, X } from 'lucide-vue-next';

interface AppState { id: 1 | 2; minimized: boolean; maximized: boolean; zIndex: number; x: number; y: number; width: number; height: number }
const props = defineProps<{ app: AppState; mobile: boolean; title: string; active: boolean }>();
const emit = defineEmits<{ focus: []; close: []; minimize: []; 'toggle-maximize': []; update: [Partial<AppState>] }>();
const dragging = ref(false);
const dragOrigin = ref({ x: 0, y: 0, left: 0, top: 0 });
const style = computed(() => props.mobile ? { zIndex: 99999 } : {
  zIndex: props.app.zIndex, left: `${props.app.maximized ? 0 : props.app.x}px`, top: `${props.app.maximized ? 0 : props.app.y}px`,
  width: props.app.maximized ? '100vw' : `${props.app.width}px`, height: props.app.maximized ? '100vh' : `${props.app.height}px`,
});
function startDrag(event: PointerEvent) {
  if (props.mobile || props.app.maximized) return;
  dragging.value = true;
  dragOrigin.value = { x: event.clientX, y: event.clientY, left: props.app.x, top: props.app.y };
  window.addEventListener('pointermove', drag);
  window.addEventListener('pointerup', stopDrag);
}
function drag(event: PointerEvent) {
  if (!dragging.value) return;
  emit('update', { x: Math.max(0, dragOrigin.value.left + event.clientX - dragOrigin.value.x), y: Math.max(0, dragOrigin.value.top + event.clientY - dragOrigin.value.y) });
  emit('focus');
}
function stopDrag() {
  dragging.value = false;
  window.removeEventListener('pointermove', drag);
  window.removeEventListener('pointerup', stopDrag);
}
onUnmounted(stopDrag);
</script>

<template>
  <article class="window" :class="{ 'window--mobile': mobile, 'window--active': active }" :style="style" @pointerdown="emit('focus')">
    <header class="window__bar" @pointerdown.stop="startDrag">
      <div class="traffic-lights">
        <button class="traffic-lights__close" title="Fechar" @click.stop="emit('close')"><X :size="11" /></button>
        <button class="traffic-lights__minimize" title="Minimizar" @click.stop="emit('minimize')"><Minus :size="11" /></button>
        <button class="traffic-lights__maximize" title="Maximizar" @click.stop="emit('toggle-maximize')"><Maximize2 :size="10" /></button>
      </div>
      <span><slot name="icon" /> {{ title }}</span>
    </header>
    <div class="window__content"><slot /></div>
  </article>
</template>
