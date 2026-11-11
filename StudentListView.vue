<template>
  <h1>Annuaire des étudiants</h1>

  <!-- TODO A : formulaire d'ajout (prenom, nom, filiere, annee, email) -->
  <form @submit.prevent="ajouterEtudiant">
    <input v-model.trim="newStudent.firstName" placeholder="Prénom" />
    <input v-model.trim="newStudent.lastName" placeholder="Nom" />
    <input v-model.trim="newStudent.filiere" placeholder="Filière" />
    <input v-model.number="newStudent.year" type="number" min="1" max="5" placeholder="Année" />
    <input v-model.trim="newStudent.email" placeholder="Email" />
    <button type="submit">Ajouter</button>
    <p v-if="erreur" class="erreur">{{ erreur }}</p>
  </form>

  <!-- TODO B : recherche (texte) + tri (select : nom / filiere) -->
  <div class="toolbar">
    <input v-model.trim="recherche" placeholder="Rechercher par nom ou prénom..." />
    <select v-model="tri">
      <option value="lastName">Trier par nom</option>
      <option value="filiere">Trier par filière</option>
    </select>
  </div>

  <!-- TODO B (suite) : bouclez sur le résultat recherché + trié -->
  <StudentCard v-for="student in etudiantsAffiches" :key="student.id" :student="student" />
</template>

<script>
import StudentCard from '../components/StudentCard.vue'
import { store } from '../store/students.js'

export default {
  name: 'StudentListView',
  components: { StudentCard },
  data() {
    return {
      store,
      recherche: '',
      tri: 'lastName',
      erreur: '',
      newStudent: { firstName: '', lastName: '', filiere: '', year: 1, email: '' }
    }
  },
  computed: {
    // TODO B : à partir de store.students, filtrez par recherche (nom ou
    // prénom contient le texte, insensible à la casse) puis triez selon
    // this.tri ('lastName' ou 'filiere')
    etudiantsAffiches() {
      const texte = this.recherche.toLowerCase()

      const filtrés = store.students.filter(student =>
        student.firstName.toLowerCase().includes(texte) ||
        student.lastName.toLowerCase().includes(texte)
      )

      return [...filtrés].sort((a, b) =>
        String(a[this.tri]).localeCompare(String(b[this.tri]), 'fr')
      )
    }
  },
  methods: {
    // TODO A : ajoutez newStudent (avec un nouvel id) à store.students,
    // validez qu'au moins prenom+nom sont renseignés avant d'ajouter
    ajouterEtudiant() {
      if (!this.newStudent.firstName || !this.newStudent.lastName) {
        this.erreur = 'Le prénom et le nom sont obligatoires.'
        return
      }

      this.erreur = ''
      const nouvelId = store.students.length
        ? Math.max(...store.students.map(s => s.id)) + 1
        : 1

      store.students.push({ ...this.newStudent, id: nouvelId })

      this.newStudent = { firstName: '', lastName: '', filiere: '', year: 1, email: '' }
    }
  }
}
</script>
