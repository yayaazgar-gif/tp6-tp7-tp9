# TP 6 – Mini-annuaire d'étudiants (Vue.js)

**Yassir Azeggarh** — TP global 2/4, navigation multi-pages.

Annuaire d'étudiants avec Vue Router : liste avec ajout, recherche et tri,
fiche détail éditable, page 404 (catch-all).

## Fonctionnalités

- **Liste (`/`)** : formulaire d'ajout validé (prénom + nom obligatoires),
  recherche insensible à la casse sur le nom/prénom, tri par nom ou filière
  (une seule propriété calculée `etudiantsAffiches` : filtre puis trie) ;
- **Fiche (`/etudiant/:id`)** : filière, année et email éditables directement
  avec `v-model` sur la donnée du store (réactive) — la liste reflète les
  changements au retour ;
- **Catch-all** : toute URL inconnue affiche la page 404.

## Lancer le projet

```bash
npm install
npm run dev
```

Puis ouvrir http://localhost:5173
