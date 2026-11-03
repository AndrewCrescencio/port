<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue';
import { Github, Linkedin, Mail, Music, User, X } from 'lucide-vue-next';
import Window from './components/Window.vue';
import Dock from './components/Dock.vue';
import ControlCenter from './components/ControlCenter.vue';

type AppId = 1 | 2;
interface OpenApp {
  id: AppId;
  minimized: boolean;
  maximized: boolean;
  zIndex: number;
  x: number;
  y: number;
  width: number;
  height: number;
}

const wallpaper = 'https://9to5mac.com/wp-content/uploads/sites/6/2025/06/Get-in-the-mood-for-macOS-Lake-Tahoe-with-these-wallpapers.jpg?quality=82&strip=all&w=1600';
const apps = [
  { id: 1 as AppId, name: 'About Me', icon: User },
  { id: 2 as AppId, name: 'iTunes', icon: Music },
];
const openApps = ref<OpenApp[]>([]);
const activeId = ref<AppId | null>(null);
const maxZ = ref(1);
const deviceType = ref<'desktop' | 'mobile'>('desktop');
const multitask = ref(false);
const multitaskIndex = ref(0);

const isMobile = computed(() => deviceType.value === 'mobile');
const activeApps = computed(() => openApps.value.filter((app) => !app.minimized).map((app) => app.id));

function resize() {
  deviceType.value = window.innerWidth < 768 ? 'mobile' : 'desktop';
}
function appName(id: AppId) {
  return apps.find((app) => app.id === id)?.name ?? '';
}
function openApp(id: AppId) {
  const existing = openApps.value.find((app) => app.id === id);
  maxZ.value += 1;
  if (existing) {
    existing.minimized = false;
    existing.zIndex = maxZ.value;
  } else {
    openApps.value.push({
      id, minimized: false, maximized: false, zIndex: maxZ.value,
      x: 60 + openApps.value.length * 20, y: 60 + openApps.value.length * 20,
      width: 800, height: 500,
    });
  }
  activeId.value = id;
  multitask.value = false;
}
function closeApp(id: AppId) {
  openApps.value = openApps.value.filter((app) => app.id !== id);
  if (activeId.value === id) activeId.value = openApps.value.at(-1)?.id ?? null;
  if (!openApps.value.length) multitask.value = false;
}
function minimizeApp(id: AppId) {
  const app = openApps.value.find((item) => item.id === id);
  if (app) app.minimized = true;
  if (activeId.value === id) activeId.value = openApps.value.find((item) => !item.minimized)?.id ?? null;
}
function focusApp(id: AppId) {
  maxZ.value += 1;
  const app = openApps.value.find((item) => item.id === id);
  if (app) {
    app.minimized = false;
    app.zIndex = maxZ.value;
  }
  activeId.value = id;
  multitask.value = false;
}
function toggleMaximize(id: AppId) {
  const app = openApps.value.find((item) => item.id === id);
  if (app) {
    app.maximized = !app.maximized;
    focusApp(id);
  }
}
function updateApp(id: AppId, patch: Partial<OpenApp>) {
  const app = openApps.value.find((item) => item.id === id);
  if (app) Object.assign(app, patch);
}
function showMultitask() {
  if (openApps.value.length) {
    multitask.value = true;
    multitaskIndex.value = Math.max(0, openApps.value.length - 1);
  }
}
function moveMultitask(direction: number) {
  multitaskIndex.value = Math.max(0, Math.min(openApps.value.length - 1, multitaskIndex.value + direction));
}

onMounted(() => {
  resize();
  window.addEventListener('resize', resize);
});
onUnmounted(() => window.removeEventListener('resize', resize));
</script>

<template>
  <main class="desktop" :style="{ backgroundImage: `url('${wallpaper}')` }">
    <div class="desktop__shade" :class="{ 'desktop__shade--multitask': multitask }" />

    <section v-if="!multitask" class="windows-layer">
      <Window
        v-for="app in openApps.filter((item) => !item.minimized)"
        :key="app.id"
        :app="app"
        :mobile="isMobile"
        :title="appName(app.id)"
        :active="app.id === activeId"
        @focus="focusApp(app.id)"
        @close="closeApp(app.id)"
        @minimize="minimizeApp(app.id)"
        @toggle-maximize="toggleMaximize(app.id)"
        @update="updateApp(app.id, $event)"
      >
        <template #icon><component :is="apps.find((item) => item.id === app.id)?.icon" :size="22" /></template>
        <div v-if="app.id === 1" class="about">
          <div class="about__photo"><img src="https://scontent.ffln4-1.fna.fbcdn.net/v/t39.30808-6/512665956_2081534118999033_7735164036701826769_n.jpg" alt="Andrew Crescencio"></div>
          <div>
            <h2><Mail :size="20" /> Contatos</h2>
            <a href="mailto:Andrew.Crescencio@email.com"><Mail :size="17" /> Andrew.Crescencio@email.com</a>
            <a href="https://www.linkedin.com/in/andrewcrescencio/" target="_blank"><Linkedin :size="17" /> linkedin.com/in/andrewcrescencio</a>
            <a href="https://github.com/andrewcrescencio" target="_blank"><Github :size="17" /> github.com/andrewcrescencio</a>
            <a href="https://wa.me/5547999500474" target="_blank">WhatsApp: (47) 99950-0474</a>
            <p>Sempre aberto a novos projetos e desafios! Vamos criar algo incrível juntos.</p>
          </div>
        </div>
        <div v-else class="itunes">
          <h2><Music :size="22" /> iTunes</h2>
          <p>Selecione uma música no player para começar.</p>
          <div class="itunes__tracks">
            <button v-for="track in ['Iron Lion Zion', 'I Shot The Sheriff', 'One Love', 'Bohemian Rhapsody']" :key="track">{{ track }}</button>
          </div>
        </div>
      </Window>
    </section>

    <button v-if="isMobile && openApps.length && !multitask" class="home-indicator" aria-label="Abrir multitarefa" @click="showMultitask" />
    <section v-if="multitask" class="multitask" @click.self="multitask = false">
      <button class="multitask__close" aria-label="Fechar multitarefa" @click="multitask = false"><X /></button>
      <div class="multitask__cards">
        <article
          v-for="(app, index) in openApps"
          :key="app.id"
          class="task-card"
          :class="{ 'task-card--active': index === multitaskIndex }"
          :style="{ transform: `translateX(${(index - multitaskIndex) * 280}px) scale(${index === multitaskIndex ? 1 : .8}) rotateY(${(index - multitaskIndex) * -15}deg)` }"
          @click="index === multitaskIndex ? focusApp(app.id) : multitaskIndex = index"
        >
          <header><component :is="apps.find((item) => item.id === app.id)?.icon" :size="24" /> {{ appName(app.id) }}</header>
          <p>{{ app.id === 1 ? 'Contatos e informações pessoais' : 'Biblioteca de músicas' }}</p>
        </article>
      </div>
      <div class="multitask__controls">
        <button @click="moveMultitask(-1)">‹</button><span>{{ multitaskIndex + 1 }} / {{ openApps.length }}</span><button @click="moveMultitask(1)">›</button>
      </div>
    </section>

    <ControlCenter :mobile="isMobile" />
    <Dock :mobile="isMobile" :active-apps="activeApps" @open="openApp" @close="closeApp" @minimize="minimizeApp" />
  </main>
</template>
