<script setup>
defineProps({
  song: Object,
  active: Boolean,
  playing: Boolean,
  liked: Boolean,
})
defineEmits(['play', 'like'])
</script>

<template>
  <article class="card" :class="{ active }" @click="$emit('play', song)">
    <div class="cover">
      <img :src="song.cover" :alt="`Artwork for ${song.title}`" loading="lazy" />
      <div class="cover-overlay">
        <span v-if="active && playing" class="bars" aria-label="Playing">
          <i></i><i></i><i></i>
        </span>
        <svg v-else viewBox="0 0 24 24" aria-hidden="true"><path d="M8 5v14l11-7z" /></svg>
      </div>
    </div>
    <div class="card-info">
      <div class="card-text">
        <h3>{{ song.title }}</h3>
        <p>{{ song.artist }}</p>
      </div>
      <button
        class="icon-btn like"
        :class="{ on: liked }"
        :aria-label="liked ? 'Unlike song' : 'Like song'"
        @click.stop="$emit('like', song)"
      >
        <svg viewBox="0 0 24 24">
          <path d="M12 21s-8-5.2-8-11a4.5 4.5 0 0 1 8-2.8A4.5 4.5 0 0 1 20 10c0 5.8-8 11-8 11z" />
        </svg>
      </button>
    </div>
  </article>
</template>
