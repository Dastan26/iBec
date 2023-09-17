<script setup>
import { onMounted, ref, computed, watch } from 'vue'
import axios from 'axios'

const apiKey = 'b855686986b8429086648564a625e6d8'
const gameList = ref([])
const selectedPlatform = ref()
const platforms = ref([])
const sortOrder = ref('asc')
const sortBy = ref('released')
const searchName = ref('')
const searchQuery = ref('')
const loadMore = ref(true)
const bottom = ref(null)

onMounted(async () => {
  try {
    const response = await loadGames(apiKey)
    gameList.value = response.data.results

    const platformNames = response.data.results.flatMap((game) =>
      game.parent_platforms.map((platform) => platform.platform.name)
    )
    platforms.value = [...new Set(platformNames)]

    const options = {
      threshold: 0.1
    }

    const observer = new IntersectionObserver(handleIntersection, options)
    observer.observe(bottom.value)
  } catch (error) {
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

const loadMoreGames = async () => {
  try {
    const response = await axios.get('https://api.rawg.io/api/games', {
      params: {
        key: apiKey,
        page: 2
      }
    })
    const newGames = response.data.results

    if (newGames.length > 0) {
      gameList.value = [...gameList.value, ...newGames]
      loadMore.value = true
    }
  } catch (error) {
    console.error('Error fetching more games', error)
  }
}

const filteredGameList = computed(() => {
  if (!selectedPlatform.value) {
    return gameList.value
  } else {
    return gameList.value.filter((game) =>
      game.parent_platforms.some((platform) => platform.platform.name === selectedPlatform.value) && game.name.toLowerCase().includes(searchQuery.value.toLowerCase())
    )
  }
})

const sortedGameList = computed(() => {
  const list = [...filteredGameList.value]

  if (sortBy.value === 'released') {
    list.sort((a, b) => {
      const dateA = new Date(a.released)
      const dateB = new Date(b.released)

      if (sortOrder.value === 'asc') {
        return dateA - dateB
      } else {
        return dateB - dateA
      }
    })
  } else if (sortBy.value === 'rating') {
    if (sortOrder.value === 'asc') {
      list.sort((a, b) => a.rating - b.rating)
    } else {
      list.sort((a, b) => b.rating - a.rating)
    }
  }

  return list
})

const toggleOrder = computed(() => (sortOrder.value === 'asc' ? 'desc' : 'asc'))
/*eslint-disable*/
watch([selectedPlatform, filteredGameList], () => {
  sortedGameList.value = sortedGameList.value
})

watch(searchName, (newSearchName) => {
  filteredGameList.value = gameList.value.filter((game) => {
    return game.name.toLowerCase().includes(newSearchName.toLowerCase())
  })
})

const handleIntersection = (entries, observer) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting && loadMore.value) {
      loadMoreGames()
      loadMore.value = false
    }
  })
};
</script>

<template>
  <div class="container">
    <header class="header">
      <input class="header-search" type="text" v-model="searchQuery" placeholder="Search for games">
      <div class="header-filters">
        <label for="platformSelect"/>
        <select id="platformSelect" v-model="selectedPlatform">
          <option value="">All Platforms</option>
          <option v-for="platform in platforms" :value="platform" :key="platform">
            {{ platform }}
          </option>
        </select>
        <label for="sortBySelect"/>
        <select id="sortBySelect" class="select-sort" v-model="sortBy">
          <option value="released">Release Date</option>
          <option value="rating">Rating</option>
        </select>
        <button class="toggle-button" @click="sortOrder = toggleOrder">
          <span v-if="sortOrder === 'asc'" class="icon up-arrow"></span>
          <span v-else class="icon down-arrow"></span>
        </button>
      </div>
    </header>

    <main class="main">
      <div v-for="game in sortedGameList" :key="game.id" class="media">
        <img class="media-img" :src="game.background_image" :alt="game.name" />
        <div class="media-info">
          <h2>{{ game.name }}</h2>
          <div class="media-info__year">
            <p>{{ game.released }}</p>
            <div class="media-info__rate"> {{ game.rating }}</div>
          </div>
        </div>
      </div>
      <div ref="bottom" v-if="sortedGameList.length > 0"></div>
    </main>
  </div>
</template>

<style lang="scss" scoped>
.header{
  margin-top: 35px;
  align-content: start;
  background: transparent;

  &-filters{
    display: flex;
    justify-content: space-between;
    margin-top: 25px;
    padding-right: 25px;

      select{
        max-width: 110px;
        padding: 4px;
        border: none;
        border-radius: 4px;
        background: #bcbcbc;;
      }
      .toggle-button {
        padding: 2px;
        border: none;
        border-radius: 4px;
        background: #bcbcbc;;
        cursor: pointer;
        transition: transform 0.3s ease;

      .icon {
        display: inline-block;
        width: 16px;
        height: 16px;
        margin-right: 5px;
        transition: transform 0.3s ease;
      }
      .up-arrow {
        background-size: cover;
        background-image: url('../assets/icons/sort.png');
      }
      .down-arrow {
        background-size: cover;
        background-image: url('../assets/icons/sort-ascending.png');
      }
    }
    .toggle-button:hover {
      transform: scale(1.1);
      .icon {
        transform: translateY(-2px);
      }
    }
  }
}
.header-search{
  padding: 8px 10px;
  border: none;
  border-radius: 12px;
  background-color: #bcbcbc;
  width: 300px;
}
.main{
  &-label{
    h1{
      margin-bottom: 5px;
    }
    p{
      margin-top: 5px;
    }
  }
  .media{
    margin-top: 20px;
    background-color: rgba(51, 51, 51, 0.733);
    &-img {
      max-width: 100%;
      padding: 5px;
      border-radius: 8px;
    }
    &-info{
      display: flex;
      align-items: center;
      justify-content: space-around;

      &__rate{
        background-color: seagreen;
        color: black;
        padding: 2px;
        border-radius: 4px;
      }
    }
  }
}

@media screen and (min-width: 768px) {
  .container {
    padding: 25px 30px;
    .header{
      display: flex;
      align-items: center;
      justify-content: flex-end;
      gap: 25px;

      &-filters{
        margin-top: 0;
        padding-right: 0;
        gap: 5px;
      }
    }

    .main{
      display: flex;
      flex-wrap: wrap;
      gap: 15px;
      justify-content: space-between;
    }
    .media{
      max-width: 45%;
      &-img{
        max-width: -webkit-fill-available;
      }
      &-info{
        padding: 15px 0;
      }
    }
  }
}

</style>
