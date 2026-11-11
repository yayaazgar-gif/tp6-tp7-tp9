<template>
  <h1>Explorateur de pays</h1>

  <p class="loading" v-show="loading">Chargement des pays...</p>
  <p class="error" v-if="error">{{ error }}</p>

  <div class="toolbar">
    <input v-model="recherche" placeholder="Rechercher un pays..." />
  </div>

  <div class="grid">
    <CountryCard v-for="country in paysAffiches" :key="country.alpha3Code" :country="country" />
  </div>
</template>

<script>
import CountryCard from '../components/CountryCard.vue'

const API_URL = 'https://countries.dev/countries'

export default {
  name: 'CountryListView',
  components: { CountryCard },
  data() {
    return {
      countries: [],
      loading: true,
      error: null,
      recherche: ''
    }
  },
  computed: {
    // TODO 2 : filtre par nom, insensible à la casse
    paysAffiches() {
      const terme = this.recherche.trim().toLowerCase()
      if (!terme) return this.countries
      return this.countries.filter(country =>
        country.name.toLowerCase().includes(terme)
      )
    }
  },
  methods: {
    // TODO 1 : chargement de tous les pays
    async chargerPays() {
      this.loading = true
      this.error = null
      try {
        const reponse = await fetch(API_URL)
        if (!reponse.ok) throw new Error(`Erreur ${reponse.status}`)
        this.countries = await reponse.json()
      } catch (e) {
        this.error = 'Impossible de charger les pays.'
      } finally {
        this.loading = false
      }
    }
  },
  mounted() {
    this.chargerPays()
  }
}
</script>
