<script setup>
defineProps({ view: String, likedCount: Number })
defineEmits(['change'])

const items = [
  { id: 'home', label: 'Home', icon: 'M3 11l9-8 9 8v10a1 1 0 0 1-1 1h-5v-7H9v7H4a1 1 0 0 1-1-1V11z' },
  { id: 'discover', label: 'Discover', icon: 'M12 2a10 10 0 1 0 0 20 10 10 0 0 0 0-20zm4 6l-2 6-6 2 2-6 6-2z' },
  { id: 'liked', label: 'Liked Songs', icon: 'M12 21s-8-5.2-8-11a4.5 4.5 0 0 1 8-2.8A4.5 4.5 0 0 1 20 10c0 5.8-8 11-8 11z' },
]
</script>

<template>
  <aside class="sidebar">
    <h1 class="brand">Musicify</h1>
    <nav>
      <button
        v-for="item in items"
        :key="item.id"
        class="nav-item"
        :class="{ active: view === item.id }"
        @click="$emit('change', item.id)"
      >
        <svg viewBox="0 0 24 24" aria-hidden="true"><path :d="item.icon" /></svg>
        <span>{{ item.label }}</span>
        <small v-if="item.id === 'liked' && likedCount">{{ likedCount }}</small>
      </button>
    </nav>
  </aside>
</template>
