<script setup lang="ts">
import { ref } from 'vue';
import { Music, User } from 'lucide-vue-next';

const props = defineProps<{ mobile: boolean; activeApps: number[] }>();
const emit = defineEmits<{ open: [1 | 2]; close: [1 | 2]; minimize: [1 | 2] }>();
const hovered = ref<number | null>(null);
const menu = ref<{ id: 1 | 2; x: number; y: number } | null>(null);
const apps = [
  { id: 2 as const, name: 'iTunes', image: 'https://upload.wikimedia.org/wikipedia/commons/9/9e/Itunes-music-app-icon.png', icon: Music },
];
function scale(index: number) {
  if (props.mobile || hovered.value === null) return 1;
  return Math.max(.9, 1.2 - Math.abs(hovered.value - index) * .25);
}
function context(event: MouseEvent, id: 1 | 2) {
  event.preventDefault();
  menu.value = { id, x: Math.min(event.clientX, window.innerWidth - 190), y: Math.min(event.clientY, window.innerHeight - 150) };
}
</script>

<template>
  <nav class="dock" :class="{ 'dock--mobile': mobile }" @click="menu = null">
    <button
      v-for="(app, index) in apps"
      :key="app.id"
      class="dock__app"
      :class="{ 'dock__app--active': activeApps.includes(app.id) }"
      :style="{ transform: `scale(${scale(index)})` }"
      :aria-label="app.name"
      @click.stop="emit('open', app.id)"
      @contextmenu="context($event, app.id)"
      @mouseenter="hovered = index"
      @mouseleave="hovered = null"
    >
      <img :src="app.image" :alt="app.name" />
      <span v-if="activeApps.includes(app.id)" />
    </button>
  </nav>
  <aside v-if="menu" class="dock-menu" :style="{ left: `${menu.x}px`, top: `${menu.y}px` }" @click.stop>
    <button @click="emit('open', menu.id); menu = null">Abrir</button>
    <button @click="emit('close', menu.id); menu = null">Fechar</button>
    <button @click="emit('minimize', menu.id); menu = null">Minimizar</button>
  </aside>
</template>
