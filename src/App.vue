<script setup>
import { ref, computed, onMounted } from 'vue'
import Sidebar from './components/Sidebar.vue'
import SongCard from './components/SongCard.vue'
import MusicPlayer from './components/MusicPlayer.vue'

// Each category is one iTunes search. Change the terms to change the music.
const CATEGORIES = [
  { name: 'Piano & Instrumental', term: 'Ludovico Einaudi' },
  { name: 'Acoustic', term: 'Jack Johnson' },
  { name: 'Jazz', term: 'Chet Baker' },
  { name: 'Gentle Lo-fi', term: 'lofi chill beats' },
  { name: 'Study & Focus', term: 'relaxing instrumental study' },
  { name: 'Feel-good Pop', term: 'Jason Mraz' },
]
const SONGS_PER_CATEGORY = 6

const songs = ref([])
const loading = ref(true)
const error = ref('')
const view = ref('home')
const query = ref('')
const liked = ref(JSON.parse(localStorage.getItem('musicify-liked') || '[]'))

const current = ref(null)
const queue = ref([])
const isPlaying = ref(false)
const player = ref(null)

async function loadCategory(category) {
  const url =
    'https://itunes.apple.com/search?media=music&entity=song&limit=25&explicit=No&term=' +
    encodeURIComponent(category.term)
  const response = await fetch(url)
  if (!response.ok) throw new Error('Request failed')
  const data = await response.json()

  const seen = new Set()
  const result = []
  for (const item of data.results) {
    const key = `${item.trackName}-${item.artistName}`.toLowerCase()
    // Keep only clean songs that really have an audio preview and artwork
    if (!item.previewUrl || !item.artworkUrl100) continue
    if (item.trackExplicitness !== 'notExplicit') continue
    if (seen.has(key)) continue
    seen.add(key)
    result.push({
      id: item.trackId,
      title: item.trackName,
      artist: item.artistName,
      album: item.collectionName,
      cover: item.artworkUrl100.replace('100x100', '400x400'),
      previewUrl: item.previewUrl,
      category: category.name,
    })
    if (result.length === SONGS_PER_CATEGORY) break
  }
  return result
}

async function loadSongs() {
  loading.value = true
  error.value = ''
  const results = await Promise.allSettled(CATEGORIES.map(loadCategory))
  const all = []
  const ids = new Set()
  for (const r of results) {
    if (r.status !== 'fulfilled') continue
    for (const song of r.value) {
      if (!ids.has(song.id)) {
        ids.add(song.id)
        all.push(song)
      }
    }
  }
  songs.value = all
  if (all.length === 0) error.value = "We couldn't load the music. Check your internet connection and try again."
  loading.value = false
}

onMounted(loadSongs)

const filtered = computed(() => {
  const text = query.value.trim().toLowerCase()
  const list = view.value === 'liked' ? songs.value.filter((s) => liked.value.includes(s.id)) : songs.value
  if (!text) return list
  return list.filter((s) => s.title.toLowerCase().includes(text) || s.artist.toLowerCase().includes(text))
})

const featured = computed(() =>
  CATEGORIES.map((c) => songs.value.find((s) => s.category === c.name)).filter(Boolean)
)

const groups = computed(() =>
  CATEGORIES.map((c) => ({ name: c.name, songs: filtered.value.filter((s) => s.category === c.name) })).filter(
    (g) => g.songs.length > 0
  )
)

function isLiked(song) {
  return liked.value.includes(song.id)
}

function toggleLike(song) {
  if (isLiked(song)) liked.value = liked.value.filter((id) => id !== song.id)
  else liked.value = [...liked.value, song.id]
  localStorage.setItem('musicify-liked', JSON.stringify(liked.value))
}

function play(song) {
  if (current.value && current.value.id === song.id) {
    player.value.togglePlay()
    return
  }
  queue.value = filtered.value.slice()
  current.value = song
}

function move(step) {
  const list = queue.value
  const index = list.findIndex((s) => s.id === current.value.id)
  // If the song is no longer in the list, start from the first one
  const next = index === -1 ? 0 : (index + step + list.length) % list.length
  current.value = list[next]
}
</script>

<template>
  <div class="app">
    <Sidebar :view="view" :liked-count="liked.length" @change="view = $event" />

    <main class="main">
      <header class="topbar">
        <div class="search">
          <svg viewBox="0 0 24 24" aria-hidden="true">
            <path d="M10 2a8 8 0 1 0 4.9 14.3l5.4 5.4 1.4-1.4-5.4-5.4A8 8 0 0 0 10 2zm0 2a6 6 0 1 1 0 12 6 6 0 0 1 0-12z" />
          </svg>
          <input v-model="query" type="search" placeholder="Search songs or artists" aria-label="Search" />
        </div>
      </header>

      <p v-if="loading" class="status">Loading music…</p>

      <div v-else-if="error" class="status">
        <p>{{ error }}</p>
        <button class="retry" @click="loadSongs">Try again</button>
      </div>

      <template v-else>
        <!-- HOME -->
        <template v-if="view === 'home'">
          <section v-if="!query && featured.length">
            <h2>Featured</h2>
            <div class="featured">
              <div
                v-for="song in featured"
                :key="song.id"
                class="feature"
                :class="{ active: current && current.id === song.id }"
                @click="play(song)"
              >
                <img :src="song.cover" :alt="`Artwork for ${song.title}`" />
                <div>
                  <h3>{{ song.title }}</h3>
                  <p>{{ song.artist }}</p>
                  <small>{{ song.category }}</small>
                </div>
              </div>
            </div>
          </section>

          <section>
            <h2>{{ query ? 'Search results' : 'All songs' }}</h2>
            <p v-if="!filtered.length" class="status">No songs match “{{ query }}”.</p>
            <div v-else class="grid">
              <SongCard
                v-for="song in filtered"
                :key="song.id"
                :song="song"
                :active="current && current.id === song.id"
                :playing="isPlaying"
                :liked="isLiked(song)"
                @play="play"
                @like="toggleLike"
              />
            </div>
          </section>
        </template>

        <!-- DISCOVER -->
        <template v-else-if="view === 'discover'">
          <p v-if="!groups.length" class="status">No songs match “{{ query }}”.</p>
          <section v-for="group in groups" :key="group.name">
            <h2>{{ group.name }}</h2>
            <div class="grid">
              <SongCard
                v-for="song in group.songs"
                :key="song.id"
                :song="song"
                :active="current && current.id === song.id"
                :playing="isPlaying"
                :liked="isLiked(song)"
                @play="play"
                @like="toggleLike"
              />
            </div>
          </section>
        </template>

        <!-- LIKED -->
        <section v-else>
          <h2>Liked Songs</h2>
          <p v-if="!liked.length" class="status">Tap the heart on a song to save it here.</p>
          <p v-else-if="!filtered.length" class="status">No liked songs match “{{ query }}”.</p>
          <div v-else class="grid">
            <SongCard
              v-for="song in filtered"
              :key="song.id"
              :song="song"
              :active="current && current.id === song.id"
              :playing="isPlaying"
              :liked="true"
              @play="play"
              @like="toggleLike"
            />
          </div>
        </section>
      </template>
    </main>

    <MusicPlayer
      ref="player"
      :song="current"
      :liked="current ? isLiked(current) : false"
      :queue-size="queue.length"
      @next="move(1)"
      @prev="move(-1)"
      @like="toggleLike"
      @playing-change="isPlaying = $event"
    />
  </div>
</template>
