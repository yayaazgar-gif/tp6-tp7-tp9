<template>
  <div class="app">
    <header class="topbar">
      <h1>Météo en direct</h1>
      <button class="unit-toggle" @click="basculerUnite" v-if="ville">
        Afficher en {{ unite === 'metric' ? '°F' : '°C' }}
      </button>
    </header>

    <div class="search">
      <input
        v-model="query"
        @input="onInputRecherche"
        type="text"
        placeholder="Rechercher une ville... (ex: Safi, Paris, Tokyo)"
      />
      <p class="hint" v-if="chargementSuggestions">Recherche...</p>
      <p class="msg-error" v-if="erreurSuggestions">{{ erreurSuggestions }}</p>

      <ul class="suggestions" v-if="suggestions.length">
        <li
          v-for="s in suggestions"
          :key="s.lat + '-' + s.lon"
          @click="choisirVille(s)"
        >
          {{ s.name }}<span v-if="s.state">, {{ s.state }}</span> ({{ s.country }})
        </li>
      </ul>
    </div>

    <div class="historique" v-if="historique.length">
      <span>Récent :</span>
      <button
        v-for="h in historique"
        :key="h.lat + '-' + h.lon"
        @click="choisirVille(h)"
      >
        {{ h.name }}
      </button>
    </div>

    <div class="meteo" v-if="ville">
      <p class="loading" v-if="chargementMeteo">Chargement de la météo...</p>
      <p class="msg-error" v-else-if="erreurMeteo">{{ erreurMeteo }}</p>

      <div v-else-if="meteoActuelle" class="courante">
        <h2>{{ ville.name }}, {{ ville.country }}</h2>

        <div class="temp-actuelle">
          <img
            :src="iconeUrl(meteoActuelle.weather[0].icon)"
            :alt="meteoActuelle.weather[0].description"
          />
          <span class="valeur">{{ Math.round(meteoActuelle.main.temp) }}°</span>
          <span class="description">
            {{ meteoActuelle.weather[0].description }}
          </span>
        </div>

        <p class="details">
          Ressenti {{ Math.round(meteoActuelle.main.feels_like) }}° ·
          Humidité {{ meteoActuelle.main.humidity }}%
        </p>
      </div>

      <div class="previsions" v-if="previsionsParJour.length">
        <div
          class="jour"
          v-for="j in previsionsParJour"
          :key="j.date"
        >
          <span class="date">{{ j.date }}</span>
          <img :src="iconeUrl(j.icone)" :alt="j.description" />
          <span class="temps">
            {{ Math.round(j.min) }}° / {{ Math.round(j.max) }}°
          </span>
        </div>
      </div>
    </div>

    <p class="empty" v-if="!ville">
      Cherchez une ville pour voir sa météo.
    </p>
  </div>
</template>

<script>
const API_KEY = import.meta.env.VITE_OWM_API_KEY
const GEO_URL = 'https://api.openweathermap.org/geo/1.0/direct'
const WEATHER_URL = 'https://api.openweathermap.org/data/2.5/weather'
const FORECAST_URL = 'https://api.openweathermap.org/data/2.5/forecast'

export default {
  name: 'App',

  data() {
    return {
      query: '',
      debounceTimer: null,
      suggestions: [],
      chargementSuggestions: false,
      erreurSuggestions: null,

      ville: null,
      historique: [],

      meteoActuelle: null,
      previsions: [],
      chargementMeteo: false,
      erreurMeteo: null,

      unite: 'metric'
    }
  },

  computed: {
    previsionsParJour() {
      const jours = {}

      for (const tranche of this.previsions) {
        const date = tranche.dt_txt.split(' ')[0]
        const heure = Number(tranche.dt_txt.split(' ')[1].slice(0, 2))

        if (!jours[date]) {
          jours[date] = {
            date,
            min: tranche.main.temp_min,
            max: tranche.main.temp_max,
            icone: tranche.weather[0].icon,
            description: tranche.weather[0].description,
            heureRepresentative: heure
          }
        } else {
          jours[date].min = Math.min(
            jours[date].min,
            tranche.main.temp_min
          )
          jours[date].max = Math.max(
            jours[date].max,
            tranche.main.temp_max
          )

          if (
            Math.abs(heure - 12) <
            Math.abs(jours[date].heureRepresentative - 12)
          ) {
            jours[date].icone = tranche.weather[0].icon
            jours[date].description = tranche.weather[0].description
            jours[date].heureRepresentative = heure
          }
        }
      }

      return Object.values(jours)
        .sort((a, b) => a.date.localeCompare(b.date))
        .map(({ heureRepresentative, ...jour }) => jour)
    }
  },

  methods: {
    iconeUrl(code) {
      return `https://openweathermap.org/img/wn/${code}@2x.png`
    },

    onInputRecherche() {
      clearTimeout(this.debounceTimer)
      this.erreurSuggestions = null

      if (!this.query.trim()) {
        this.suggestions = []
        this.chargementSuggestions = false
        return
      }

      this.debounceTimer = setTimeout(() => {
        this.rechercherVilles()
      }, 400)
    },

    async rechercherVilles() {
      const recherche = this.query.trim()

      if (!recherche) {
        this.suggestions = []
        return
      }

      this.chargementSuggestions = true
      this.erreurSuggestions = null

      try {
        if (!API_KEY || API_KEY === 'votre_cle_ici') {
          throw new Error(
            'Ajoutez une clé API valide dans le fichier .env.'
          )
        }

        const url =
          `${GEO_URL}?q=${encodeURIComponent(recherche)}` +
          `&limit=5&appid=${API_KEY}`

        const reponse = await fetch(url)

        if (reponse.status === 401) {
          throw new Error(
            'Clé API invalide ou absente. Vérifiez votre fichier .env.'
          )
        }

        if (reponse.status === 429) {
          throw new Error('Quota API dépassé. Réessayez plus tard.')
        }

        if (!reponse.ok) {
          throw new Error(`Erreur du service météo (${reponse.status}).`)
        }

        const villes = await reponse.json()
        this.suggestions = villes

        if (villes.length === 0) {
          this.erreurSuggestions =
            'Aucune ville trouvée pour cette recherche.'
        }
      } catch (erreur) {
        this.suggestions = []
        this.erreurSuggestions =
          erreur.message ||
          'Impossible de rechercher les villes. Vérifiez votre connexion.'
      } finally {
        this.chargementSuggestions = false
      }
    },

    choisirVille(ville) {
      this.suggestions = []
      this.query = ''
      this.erreurSuggestions = null
      this.ville = ville

      this.historique = [
        ville,
        ...this.historique.filter(
          (element) =>
            element.lat !== ville.lat || element.lon !== ville.lon
        )
      ].slice(0, 5)

      this.chargerMeteo()
    },

    async chargerMeteo() {
      if (!this.ville) return

      this.chargementMeteo = true
      this.erreurMeteo = null

      try {
        if (!API_KEY || API_KEY === 'votre_cle_ici') {
          throw new Error(
            'Ajoutez une clé API valide dans le fichier .env.'
          )
        }

        const { lat, lon } = this.ville
        const parametres =
          `lat=${lat}&lon=${lon}&appid=${API_KEY}` +
          `&units=${this.unite}&lang=fr`

        const [reponseMeteo, reponsePrevisions] = await Promise.all([
          fetch(`${WEATHER_URL}?${parametres}`),
          fetch(`${FORECAST_URL}?${parametres}`)
        ])

        for (const reponse of [reponseMeteo, reponsePrevisions]) {
          if (reponse.status === 401) {
            throw new Error(
              'Clé API invalide ou absente. Vérifiez votre fichier .env.'
            )
          }

          if (reponse.status === 429) {
            throw new Error('Quota API dépassé. Réessayez plus tard.')
          }

          if (!reponse.ok) {
            throw new Error(`Erreur du service météo (${reponse.status}).`)
          }
        }

        const [meteo, previsions] = await Promise.all([
          reponseMeteo.json(),
          reponsePrevisions.json()
        ])

        this.meteoActuelle = meteo
        this.previsions = previsions.list
      } catch (erreur) {
        this.meteoActuelle = null
        this.previsions = []
        this.erreurMeteo =
          erreur.message ||
          'Impossible de charger la météo. Vérifiez votre connexion.'
      } finally {
        this.chargementMeteo = false
      }
    },

    basculerUnite() {
      this.unite =
        this.unite === 'metric' ? 'imperial' : 'metric'

      this.chargerMeteo()
    }
  }
}
</script>