<script setup>
import { ref, watch, onMounted } from 'vue'

const props = defineProps({
  song: Object,
  liked: Boolean,
  queueSize: Number,
})
const emit = defineEmits(['next', 'prev', 'like', 'playing-change'])

const audio = ref(null)
const playing = ref(false)
const currentTime = ref(0)
const duration = ref(0)
const volume = ref(0.8)
const muted = ref(false)
const message = ref('')

onMounted(() => {
  audio.value.volume = volume.value
})

// Load and play whenever a new song is chosen
watch(
  () => props.song,
  (song) => {
    if (!song) return
    message.value = ''
    currentTime.value = 0
    duration.value = 0
    audio.value.src = song.previewUrl
    start()
  }
)

watch(volume, (v) => {
  audio.value.volume = v
  if (v > 0) muted.value = false
})
watch(muted, (m) => {
  audio.value.muted = m
})

async function start() {
  try {
    await audio.value.play()
  } catch (err) {
    if (err.name === 'NotAllowedError') {
      message.value = 'Your browser blocked autoplay. Press play to start.'
    } else if (err.name !== 'AbortError') {
      message.value = "Sorry, this track can't be played right now."
    }
  }
}

function togglePlay() {
  if (!props.song) return
  if (audio.value.paused) start()
  else audio.value.pause()
}

function skip(direction) {
  // With a single song in the list, "next" and "previous" just restart it
  if (props.queueSize <= 1) {
    audio.value.currentTime = 0
    start()
  } else {
    emit(direction)
  }
}

function seek(event) {
  audio.value.currentTime = Number(event.target.value)
}

function onPlay() {
  playing.value = true
  message.value = ''
  emit('playing-change', true)
}

function onPause() {
  playing.value = false
  emit('playing-change', false)
}

function onError() {
  if (props.song) message.value = "Sorry, this track can't be played right now."
}

function formatTime(seconds) {
  if (!Number.isFinite(seconds)) return '0:00'
  const m = Math.floor(seconds / 60)
  const s = Math.floor(seconds % 60)
  return `${m}:${String(s).padStart(2, '0')}`
}

defineExpose({ togglePlay })
</script>

<template>
  <footer class="player">
    <audio
      ref="audio"
      preload="metadata"
      @timeupdate="currentTime = audio.currentTime"
      @loadedmetadata="duration = audio.duration"
      @ended="skip('next')"
      @play="onPlay"
      @pause="onPause"
      @error="onError"
    ></audio>

    <div class="player-song">
      <template v-if="song">
        <img :src="song.cover" :alt="`Artwork for ${song.title}`" />
        <div class="player-text">
          <strong>{{ song.title }}</strong>
          <span>{{ song.artist }}</span>
        </div>
        <button
          class="icon-btn like"
          :class="{ on: liked }"
          :aria-label="liked ? 'Unlike song' : 'Like song'"
          @click="$emit('like', song)"
        >
          <svg viewBox="0 0 24 24">
            <path d="M12 21s-8-5.2-8-11a4.5 4.5 0 0 1 8-2.8A4.5 4.5 0 0 1 20 10c0 5.8-8 11-8 11z" />
          </svg>
        </button>
      </template>
      <p v-else class="player-empty">Pick a song to start listening</p>
    </div>

    <div class="player-center">
      <div class="controls">
        <button class="icon-btn" aria-label="Previous song" :disabled="!song" @click="skip('prev')">
          <svg viewBox="0 0 24 24"><path d="M6 5h2v14H6zM20 5v14L9 12z" /></svg>
        </button>
        <button class="play-btn" :aria-label="playing ? 'Pause' : 'Play'" :disabled="!song" @click="togglePlay">
          <svg v-if="playing" viewBox="0 0 24 24"><path d="M7 5h4v14H7zM13 5h4v14h-4z" /></svg>
          <svg v-else viewBox="0 0 24 24"><path d="M8 5v14l11-7z" /></svg>
        </button>
        <button class="icon-btn" aria-label="Next song" :disabled="!song" @click="skip('next')">
          <svg viewBox="0 0 24 24"><path d="M16 5h2v14h-2zM4 5l11 7-11 7z" /></svg>
        </button>
      </div>

      <div class="progress">
        <span>{{ formatTime(currentTime) }}</span>
        <input
          type="range"
          min="0"
          :max="duration || 0"
          step="0.1"
          :value="currentTime"
          :disabled="!song"
          :style="{ '--fill': duration ? (currentTime / duration) * 100 + '%' : '0%' }"
          aria-label="Seek"
          @input="seek"
        />
        <span>{{ formatTime(duration) }}</span>
      </div>
      <p v-if="message" class="player-message">{{ message }}</p>
      <p v-else-if="song" class="player-note">30-second preview</p>
    </div>

    <div class="player-volume">
      <button class="icon-btn" :aria-label="muted ? 'Unmute' : 'Mute'" @click="muted = !muted">
        <svg v-if="muted || volume === 0" viewBox="0 0 24 24">
          <path d="M3 9v6h4l5 4V5L7 9H3zm13.6 3l2.7-2.7-1.4-1.4-2.7 2.7-2.7-2.7-1.4 1.4 2.7 2.7-2.7 2.7 1.4 1.4 2.7-2.7 2.7 2.7 1.4-1.4z" />
        </svg>
        <svg v-else viewBox="0 0 24 24">
          <path d="M3 9v6h4l5 4V5L7 9H3zm13.5 3a4.5 4.5 0 0 0-2.5-4v8a4.5 4.5 0 0 0 2.5-4z" />
        </svg>
      </button>
      <input
        type="range"
        min="0"
        max="1"
        step="0.01"
        v-model.number="volume"
        :style="{ '--fill': (muted ? 0 : volume * 100) + '%' }"
        aria-label="Volume"
      />
    </div>
  </footer>
</template>
