<script setup>
import { onMounted, ref, computed } from 'vue'
import axios from 'axios'
import HeaderComponent from '@/components/HeaderComponent.vue'

const apiKey = 'b855686986b8429086648564a625e6d8'
const gameList = ref([])
const selectedPlatform = ref('')
const platforms = ref([])

onMounted(async () => {
  try {
    const response = await loadGames(apiKey)
    gameList.value = response.data.results
    platforms.value = [...new Set(response.data.results.map((game) => game.platforms))]
  } catch (error){
    console.error('Error fetching', error)
  }
})

const loadGames = async (apiKey) => {
  const response = await axios.get('https://api.rawg.io/api/games', {
    params: {
      key: apiKey
    }
  })
  return response
}

const filteredGameList = computed(() => {
  if (!selectedPlatform.value){
    return gameList.value
  } else {
    return gameList.value.filter((game) =>
      game.platforms.includes(selectedPlatform.value)
    )
  }
})

</script>

<template>
  <header-component />
  <main>
    <div class="main-label">
      <h1>New and trending</h1>
      <p>Based on player counts and release date</p>
    </div>
    <div class="main-filters">
      <label for="platformSelect">Select Platform:</label>
      <select id="platformSelect" v-model="selectedPlatform">
        <option value="">All Platforms</option>
        <option v-for="platform in platforms" :value="platform" :key="platform">
          {{ platform }}w
        </option>
      </select>
    </div>
    <div v-for="game in filteredGameList" :key="game.id" class="media-item">
      <img class="game-img" :src="game.background_image" :alt="game.name" />
      <h2>{{ game.name }}</h2>
      <p>Rating: {{ game.rating }}</p>
      <p>Release Date: {{ game.released }}</p>
    </div>
  </main>
</template>

<style lang="scss" scoped>
  .game-img {
    max-width: 100%;
  }
</style>
