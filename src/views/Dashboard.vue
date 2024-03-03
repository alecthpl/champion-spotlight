<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

interface Champ {
  id: string
  name: string
  tags: string[]
}

const champions = ref<Champ[]>([])
const tags = ref<string[]>([])
const searchTerm = ref<string>('')
const selectedTag = ref<string>('')

const filteredChamps = computed(() => {
  return champions.value?.filter(champ => {
    const includesSearch = champ.name.toLowerCase().includes(searchTerm.value.toLowerCase())
    const includesOption = !selectedTag.value || champ.tags.includes(selectedTag.value)

    return includesSearch && includesOption
  })
})

const clearFilters = () => {
  selectedTag.value = ''
  searchTerm.value = ''
}

onMounted(async () => {
  try {
    const res = await fetch('/data/champion.json')
    const { data } = await res.json()

    // get unique array of tags
    const uniqueTags = new Set<string>()
    Object.keys(data).forEach(key => {
      data[key].tags.forEach((tag: string) => {
        uniqueTags.add(tag)
      })
    })
    tags.value = [...uniqueTags]

    champions.value = Object.keys(data).map(key => data[key])
  } catch (err) {
    console.log(err)
  }
})
</script>

<template>
  <header>
    <h1>Champion Spotlight</h1>
    <div class="search">
      <input v-model="searchTerm" type="text" placeholder="Enter a champion's name..." />
      <div>
        <select v-model="selectedTag">
          <option disabled selected value="">Filter by Role</option>
          <option v-for="tag in tags" :key="tag" :value="tag">
            {{ tag }}
          </option>
        </select>
        <button @click="clearFilters">Clear</button>
      </div>
    </div>
  </header>
  <div v-if="filteredChamps.length" class="champs">
    <div v-for="champ in filteredChamps" :key="champ.id">
      <RouterLink :to="`/champion/${champ.id}`">
        <div :style="`background-image: url(/data/champion-images/tiles/${champ.id}_0.jpg)`" class="champ-card">
          <h5>{{ champ.name }}</h5>
        </div>
      </RouterLink>
    </div>
  </div>
  <div v-else class="no-results">
    <div>
      <h2>No Results</h2>
      <p>Clear filters and try again</p>
    </div>
  </div>
</template>

<style scoped>
header {
  display: flex;
  align-items: center;
  justify-content: end;
  padding: 20px 40px 0;
}

header h1 {
  font-size: 20px;
  margin-right: auto;
  text-transform: uppercase;
}

header button {
  cursor: pointer;
  border: 0;
  outline: 0;
  padding: 8px;
}

.search {
  display: flex;
  gap: 16px;
}

input {
  width: 300px;
}

select {
  width: 150px;
}

.champs {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
  padding: 40px;
}

.champs > div {
  transition: 0.15s;
}

.champs > div:hover {
  filter: brightness(125%);
}

.champ-card {
  position: relative;
  display: flex;
  height: 300px;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}

img {
  display: block;
  border-radius: 4px;
}

h5 {
  width: 100%;
  margin: auto 0 0;
  padding: 12px;
  background: rgba(30, 30, 30, 0.8);
}

.no-results {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 40px;
  height: calc(100vh - 57.5px);
}

.no-results > div {
  text-align: center;
}

@media (max-width: 875px) {
  header {
    flex-direction: column;
    gap: 16px;
  }

  header h1 {
    margin-right: unset;
  }
}

@media (max-width: 605px) {
  .search {
    flex-direction: column;
    width: 100%;
  }

  input {
    width: 100%;
  }
}
</style>