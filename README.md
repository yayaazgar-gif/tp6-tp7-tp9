# TP 9 — Météo en direct

Application Vue 3 et Vite permettant de rechercher une ville, consulter la météo actuelle et afficher les prévisions sur cinq jours. Les données météo sont fournies par OpenWeatherMap.

## Installation

1. Installez Node.js.
2. Dans le dossier du projet, installez les dépendances :
   ```sh
   npm install
   ```
3. Copiez `.env.example` sous le nom `.env` et renseignez votre clé OpenWeatherMap :
   ```env
   VITE_OWM_API_KEY=votre_cle_openweathermap
   ```
4. Lancez l’application :
   ```sh
   npm run dev
   ```

La clé utilisée par une application Vite est exposée au navigateur. N’utilisez pas cette configuration pour une clé qui doit rester confidentielle.
