<template>
  <p class="loading" v-show="loading">Chargement...</p>
  <p class="error" v-if="error">{{ error }}</p>

  <div v-if="country">
    <img :src="country.flags.svg" :alt="`Drapeau ${country.name}`" style="width: 200px;" />
    <h1>{{ country.name }}</h1>
    <p>Nom natif : {{ country.nativeName || '—' }}</p>
    <p>Capitale : {{ country.capital || '—' }}</p>
    <p>Région : {{ country.region }}</p>
    <p>Population : {{ country.population.toLocaleString() }}</p>

    <!-- Bonus : liens vers les pays voisins (même vue, rechargée via watch) -->
    <p v-if="country.borders && country.borders.length">
      Voisins :
      <router-link
        v-for="code in country.borders"
        :key="code"
        :to="`/pays/${code}`"
        style="margin-right: 8px;"
      >{{ code }}</router-link>
    </p>
  </div>

  <button @click="$router.back()">Retour</button>
</template>

<script>
export default {
  name: 'CountryDetailView',
  data() {
    return {
      country: null,
      loading: true,
      error: null
    }
  },
  methods: {
    // TODO 3 : chargement d'un pays par son code
    async chargerPays() {
      this.loading = true
      this.error = null
      this.country = null
      try {
        const code = this.$route.params.code
        const reponse = await fetch(`https://countries.dev/alpha/${code}`)
        if (!reponse.ok) throw new Error(`Erreur ${reponse.status}`)
        const donnees = await reponse.json()
        // La réponse peut être un objet unique ou un tableau d'un seul élément
        this.country = Array.isArray(donnees) ? donnees[0] : donnees
        if (!this.country) throw new Error('Pays introuvable')
      } catch (e) {
        this.error = 'Impossible de charger ce pays.'
      } finally {
        this.loading = false
      }
    }
  },
  watch: {
    // Bonus : recharger quand le code change sans démontage du composant
    '$route.params.code'(nouveau) {
      if (nouveau) this.chargerPays()
    }
  },
  mounted() {
    this.chargerPays()
  }
}
</script>
