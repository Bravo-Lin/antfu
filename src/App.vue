<script setup lang="ts">
useHead({
  meta: [
    { property: 'og:title', content: '2222' },
    { property: 'og:image', content: 'https://avatars.githubusercontent.com/u/43205138?v=4' },
    { name: 'description', content: 'Bravo Lin\'s Portfolio' },
  ],
})
</script>

<template>
  <NavBar />
  <main class="px-7 py-10">
    <router-view />
    <Footer />
  </main>
</template>

