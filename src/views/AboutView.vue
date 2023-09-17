<template>
  <div>
    <div class="game-details">
      <img class="game-img" :src="selectedGame.background_image_additional" alt="image">
      <h1>{{ selectedGame.name }}</h1>
      <p>{{ selectedGame.description_raw }}</p>
      <a>{{ selectedGame.website }}</a>
    </div>
  </div>
</template>

<script setup>
import axios from 'axios'
import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'

const selectedGame = ref({})
const apiKey = 'b855686986b8429086648564a625e6d8'
const route = useRoute()

onMounted(async () => {
  const gameId = route.params.gameId
  try {
    const response = await axios.get(`https://api.rawg.io/api/games/${gameId}`, {
      params: {
        key: apiKey
      }
    })
    if (response.data && response.data.id) {
      selectedGame.value = response.data
      console.log(selectedGame.value)
    } else {
      console.error('Game details not found')
    }
  } catch (error) {
    console.error('Error fetching game details', error)
  }
})
</script>

<style scoped>
.game-img{
  max-width: 100%;
}
</style>
