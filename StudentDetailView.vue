<template>
  <div v-if="student">
    <h1>{{ student.firstName }} {{ student.lastName }}</h1>

    <!-- TODO D : édition directe des champs avec v-model sur "student" -->
    <p>
      Filière :
      <input v-model.trim="student.filiere" />
    </p>
    <p>
      Année :
      <input v-model.number="student.year" type="number" min="1" max="5" />
    </p>
    <p>
      Email :
      <input v-model.trim="student.email" type="email" />
    </p>
  </div>
  <div v-else>
    <p>Étudiant introuvable.</p>
  </div>

  <button @click="$router.back()">Retour</button>
</template>

<script>
import { store } from '../store/students.js'

export default {
  name: 'StudentDetailView',
  computed: {
    // TODO C : retrouvez dans store.students l'étudiant dont l'id
    // correspond au paramètre de route (rappel du TP4 : $route.params.id
    // est une chaîne)
    student() {
      const id = Number(this.$route.params.id)
      return store.students.find(student => student.id === id)
    }
  }
}
</script>
