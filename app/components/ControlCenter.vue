<script setup lang="ts">
import { computed, onUnmounted, ref } from 'vue';
import { Bluetooth, Monitor, Moon, Pause, Play, SkipBack, SkipForward, Wifi } from 'lucide-vue-next';

defineProps<{ mobile: boolean }>();
const tracks: [string, string][] = [
  ['Iron Lion Zion', 'Boby Marley'], ['I Shot The Sheriff', 'Boby Marley'],
  ['One Love', 'Boby Marley'], ['Bohemian Rhapsody', 'Queen'],
];
const defaultTrack: [string, string] = ['Iron Lion Zion', 'Boby Marley'];
const defaultSource = 'https://github.com/PassaUmDollar/itunes_audio/blob/master/audio/01.%20Iron%20Lion%20Zion.mp3?raw=true';
const index = ref(0);
const playing = ref(false);
const volume = ref(60);
const audio = ref<HTMLAudioElement>();
const sources = [
  'https://github.com/PassaUmDollar/itunes_audio/blob/master/audio/01.%20Iron%20Lion%20Zion.mp3?raw=true',
  'https://github.com/PassaUmDollar/itunes_audio/blob/master/audio/04.%20I%20Shot%20The%20Sheriff.mp3?raw=true',
  'https://github.com/PassaUmDollar/itunes_audio/blob/master/audio/06.%20One%20Love.mp3?raw=true',
  'https://github.com/PassaUmDollar/itunes_audio/blob/master/audio/07.%20Bohemian%20Rhapsody.mp3?raw=true',
];
const track = computed<[string, string]>(() => tracks[index.value] ?? defaultTrack);
function play() {
  if (!audio.value) return;
  audio.value.src = sources[index.value] ?? defaultSource;
  audio.value.volume = volume.value / 100;
  audio.value.play().then(() => playing.value = true).catch(() => playing.value = false);
}
function toggle() {
  if (!audio.value) return;
  if (playing.value) { audio.value.pause(); playing.value = false; } else play();
}
function next(step = 1) { index.value = (index.value + step + tracks.length) % tracks.length; playing.value = false; play(); }
onUnmounted(() => audio.value?.pause());
</script>

<template>
  <section class="control-center">
    <div class="control-center__toggles"><button><Wifi /></button><button><Bluetooth /></button><button><Moon /></button><button><Monitor /></button></div>
    <div class="control-center__player">
      <div class="album-art">♪</div>
      <div class="control-center__track"><strong>{{ track[0] }}</strong><small>{{ track[1] }}</small></div>
      <div class="player-actions"><button @click="next(-1)"><SkipBack :size="18" /></button><button @click="toggle"><Pause v-if="playing" :size="18" /><Play v-else :size="18" /></button><button @click="next()"><SkipForward :size="18" /></button></div>
    </div>
    <label>Volume <input v-model="volume" type="range" min="0" max="100" @input="audio && (audio.volume = volume / 100)" /></label>
    <audio ref="audio" @ended="next()" />
  </section>
</template>
