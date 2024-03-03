<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()

interface Champ {
  id: string
  name: string
  title: string
  lore: string
  passive: {
    name: string
    description: string
    image: {
      full: string
    }
  }
  spells: {
    id: string
    name: string
    description: string
    image: {
      full: string
    }
  }[]
  skins: {
    id: string
    num: number
  }[]
  tags: string[]
}

const champ = ref<Champ>({
  id: '',
  name: '',
  title: '',
  lore: '',
  passive: {
    name: '',
    description: '',
    image: {
      full: ''
    }
  },
  spells: [],
  skins: [],
  tags: [],
})

const selectedSkin = ref<number>(0)

const removeTag = (text: string) => {
  return text?.replace(/<br\s*\/?>/i, '')
}

onMounted(async () => {
  const id: any = route.params?.id || ''

  const res = await fetch(`/data/champion-data/${id}.json`)
  const { data } = await res.json()
  
  champ.value = data[id]

  const randomIndex = Math.floor(Math.random() * champ.value.skins.length)
  selectedSkin.value = champ.value.skins[randomIndex].num
})
</script>

<template>
  <RouterLink to="/" class="back-button">back</RouterLink>
  <div class="hero" :style="`background-image: url('/data/champion-images/centered/${champ.id}_${selectedSkin}.jpg')`">
    <div class="content">
      <div class="info" :style="`background-image: url('/data/champion-images/centered/${champ.id}_${selectedSkin}.jpg')`">
        <h1>{{ champ.name }}</h1>
        <h2>{{ champ.title }}</h2>
        <h6>{{ champ.tags?.join(', ') }}</h6>
        <p>{{ champ.lore }}</p>
        <div class="passive">
          <div class="spell">
            <div class="img">
              <img :src="`/data/passive/${champ.passive?.image.full}`" />
            </div>
            <div class="spell-info">
              <h6>{{ champ.passive?.name }}</h6>
              <p v-html="removeTag(champ.passive?.description)"></p>
            </div>
          </div>
        </div>
      </div>
      <div class="spells">
        <div v-for="spell in champ.spells" :key="spell.name" class="spell">
          <div class="img">
            <img :src="`/data/spell/${spell.image.full}`" />
          </div>
          <div class="spell-info">
            <h6>{{ spell.name }}</h6>
            <p v-html="removeTag(spell.description)"></p>
          </div>
        </div>
      </div>
    </div>
    <div class="skins">
      <div v-for="skin in champ?.skins" :key="skin.id" :class="{ 'selected': selectedSkin === skin.num }">
        <img @click="selectedSkin = skin.num" :src="`/data/champion-images/splash/${champ.id}_${skin.num}.jpg`" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.back-button {
  position: fixed;
  top: 8px;
  left: 8px;
  padding: 8px;
  z-index: 10;
}

.selected {
  outline: 2px solid var(--accent);
}

.hero {
  display: flex;
  flex-direction: column;
  justify-content: center;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  min-height: 100vh;
}

.content {
  margin: auto 0;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 60px;
}

.tags {
  display: flex;
}

.info {
  max-width: 600px;
}

.info h1 {
  font-size: 100px;
}

.info h2 {
  font-size: 40px;
  margin: -16px 0 16px;
}

.info h6 {
  font-size: 16px;
  margin-bottom: 16px;
}

.spells,
.passive {
  display: flex;
  flex-direction: column;
  gap: 16px;
  max-width: 400px;
  background: rgba(60, 60, 60, 0.4);
  padding: 32px;
  border-radius: 16px;
}

.spells {
  max-height: 600px;
  overflow-y: auto;
}

.passive {
  max-width: none;
  padding: 16px;
  margin-top: 16px;
}

.passive h6 {
  margin-bottom: 8px;
}

.spell {
  display: flex;
  align-items: center;
  gap: 12px;
}

.spell img {
  width: 75px;
  height: 75px;
  border-radius: 50%;
  border: 2px solid var(--accent);
  padding: 2px;
}

.spell .info {
  max-width: 100px;
}

.spell p {
  overflow: hidden;
  text-overflow: ellipsis;
  font-size: 12px;
  line-clamp: 2;
}

.skins {
  display: flex;
  gap: 16px;
  background: rgba(60, 60, 60, 0.4);
  padding: 16px;
  min-height: 179.52px;
  overflow-x: auto;
}

.skins img {
  width: 250px;
  cursor: pointer;
  transition: 0.15s;
}
.skins img:hover {
  opacity: 0.6;
}

@media (min-width: 1126px) {
  .info {
    background-image: none !important;
  }
}

@media (max-width: 1125px) {
  .hero {
    background-image: none !important;
  }

  .content {
    flex-direction: column;
    padding: 0;
  }

  .info {
    max-width: unset;
    padding: 40px;
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
  }

  .spells {
    margin: 40px;
    max-width: unset;
    max-height: unset;
    overflow-y: unset;
  }

  .skins img {
    width: 450px;
  }
}

@media (max-width: 500px) {
  .info h1 {
    font-size: 50px;
  }

  .info h2 {
    font-size: 22px;
    margin-top: 0;
  }
}
</style>