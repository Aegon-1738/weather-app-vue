<!-- src/App.vue -->
<template>
  <div class="app-container">

    <!-- ================= HEADER ================= -->
    <header class="app-header">
      <div class="logo">☀️</div>
      <div>
        <h1>Weather App</h1>
        <p class="tagline">Check the weather anywhere in the world</p>
      </div>
    </header>

    <!-- ================= MAIN ================= -->
    <main class="app-main">

      <section class="hero">
        <h2>Search a city</h2>
        <p>Enter a city name to get its current weather.</p>
      </section>

      <!-- SearchBar : v-model interne + @search vers le parent -->
      <SearchBar @search="handleSearch" :disabled="loading" />

      <!-- Historique des recherches : utilise v-for -->
      <ul v-if="history.length > 0" class="history" aria-label="Recent searches">
        <li v-for="(item, index) in history" :key="index" class="history-item">
          <button
            type="button"
            class="history-btn"
            @click="handleSearch(item)"
            :aria-label="`Search again for ${item}`"
          >
            🕘 {{ item }}
          </button>
        </li>
      </ul>

      <!-- ================= ETATS ================= -->

      <!-- v-if : affiche le message d'erreur seulement s'il existe -->
      <div v-if="error" class="error-box" role="alert">
        ⚠️ {{ error }}
      </div>

      <!-- LoadingMessage affiché uniquement pendant le chargement -->
      <LoadingMessage v-if="loading" />

      <!-- WeatherCard affichée uniquement si pas de loading et weather présent -->
      <WeatherCard
        v-else-if="weather"
        :weather="weather"
      />

      <!-- v-else : état initial (aucune recherche) -->
      <div v-else class="empty-state">
        <p>🌍 Enter a city above to see the weather.</p>
      </div>

      <!-- v-show : petit statut toujours présent dans le DOM, masqué par CSS -->
      <p class="status-line" v-show="searchStatus">
        Status : {{ searchStatus }}
      </p>

    </main>

    <footer class="app-footer">
      <p>Powered by <strong>Open-Meteo API</strong> · Built with Vue.js 3</p>
    </footer>
  </div>
</template>

<script>
import SearchBar from './components/SearchBar.vue'
import WeatherCard from './components/WeatherCard.vue'
import LoadingMessage from './components/LoadingMessage.vue'

export default {
  name: 'App',

  components: {
    SearchBar,
    WeatherCard,
    LoadingMessage
  },

  data() {
    return {
      cityName: '',        // dernière ville recherchée
      weather: null,       // objet météo formaté
      loading: false,      // état de chargement
      error: '',           // message d'erreur
      searchStatus: '',    // texte de statut (utilisé par v-show)
      history: []          // historique des recherches (v-for)
    }
  },

  methods: {
    /**
     * Méthode appelée quand SearchBar émet "search"
     * ou quand l'utilisateur clique sur l'historique.
     */
    async handleSearch(city) {
      // 1) Validation du champ vide
      const cleanCity = (city || '').trim()
      if (!cleanCity) {
        this.error = 'Please enter a city name.'
        this.weather = null
        this.searchStatus = 'Idle'
        return
      }

      // 2) Réinitialisation de l'état
      this.cityName = cleanCity
      this.loading = true
      this.error = ''
      this.weather = null
      this.searchStatus = `Searching for ${cleanCity}...`

      try {
        // 3) Géocodage : ville -> latitude / longitude
        const geoUrl =
          `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(cleanCity)}&count=1&language=en&format=json`

        const geoRes = await fetch(geoUrl)

        if (!geoRes.ok) {
          throw new Error('NETWORK')
        }

        const geoData = await geoRes.json()

        // 4) Ville inexistante
        if (!geoData.results || geoData.results.length === 0) {
          this.error = 'City not found. Please check the spelling and try again.'
          this.searchStatus = 'Not found'
          return
        }

        const place = geoData.results[0]
        const { latitude, longitude, name, country } = place

        // 5) Météo : latitude / longitude -> current weather
        const weatherUrl =
          `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}` +
          `&current=temperature_2m,relative_humidity_2m,wind_speed_10m,weather_code` +
          `&timezone=auto`

        const weatherRes = await fetch(weatherUrl)

        if (!weatherRes.ok) {
          throw new Error('NETWORK')
        }

        const weatherData = await weatherRes.json()
        const current = weatherData.current

        // 6) Formatage des données pour WeatherCard
        this.weather = {
          city: name,
          country: country || '',
          temperature: Math.round(current.temperature_2m),
          humidity: current.relative_humidity_2m,
          windSpeed: Math.round(current.wind_speed_10m),
          condition: this.getWeatherCondition(current.weather_code),
          icon: this.getWeatherIcon(current.weather_code)
        }

        this.searchStatus = `Weather loaded for ${name}`

        // 7) Historique : évite les doublons, garde les 5 derniers
        this.history = [
          name,
          ...this.history.filter((c) => c !== name)
        ].slice(0, 5)

      } catch (err) {
        // Erreur réseau / API
        this.error = 'Unable to retrieve weather data. Please try again later.'
        this.searchStatus = 'Error'
      } finally {
        // Toujours exécuté : on arrête le loading
        this.loading = false
      }
    },

    /**
     * Convertit un code WMO Open-Meteo en description lisible.
     * Référence : https://open-meteo.com/en/docs
     */
    getWeatherCondition(code) {
      const map = {
        0: 'Clear Sky',
        1: 'Mainly Clear',
        2: 'Partly Cloudy',
        3: 'Overcast',
        45: 'Fog',
        48: 'Depositing Rime Fog',
        51: 'Light Drizzle',
        53: 'Moderate Drizzle',
        55: 'Dense Drizzle',
        56: 'Light Freezing Drizzle',
        57: 'Dense Freezing Drizzle',
        61: 'Slight Rain',
        63: 'Moderate Rain',
        65: 'Heavy Rain',
        66: 'Light Freezing Rain',
        67: 'Heavy Freezing Rain',
        71: 'Slight Snow',
        73: 'Moderate Snow',
        75: 'Heavy Snow',
        77: 'Snow Grains',
        80: 'Slight Rain Showers',
        81: 'Moderate Rain Showers',
        82: 'Violent Rain Showers',
        85: 'Slight Snow Showers',
        86: 'Heavy Snow Showers',
        95: 'Thunderstorm',
        96: 'Thunderstorm with Slight Hail',
        99: 'Thunderstorm with Heavy Hail'
      }
      return map[code] || 'Unknown'
    },

    /**
     * Associe un emoji à chaque code météo.
     */
    getWeatherIcon(code) {
      if (code === 0) return '☀️'
      if (code === 1 || code === 2) return '🌤️'
      if (code === 3) return '☁️'
      if (code === 45 || code === 48) return '🌫️'
      if (code >= 51 && code <= 57) return '🌦️'
      if (code >= 61 && code <= 67) return '🌧️'
      if (code >= 71 && code <= 77) return '❄️'
      if (code >= 80 && code <= 82) return '🌧️'
      if (code === 85 || code === 86) return '🌨️'
      if (code >= 95) return '⛈️'
      return '🌡️'
    }
  }
}
</script>

<style scoped>
.app-container {
  width: 100%;
  max-width: 720px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

/* Header */
.app-header {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 20px;
  background: var(--card-bg);
  border: 1px solid var(--card-border);
  border-radius: var(--radius);
  backdrop-filter: blur(12px);
  box-shadow: var(--shadow);
}

.logo {
  font-size: 2.4rem;
}

.app-header h1 {
  font-size: 1.5rem;
  letter-spacing: 0.5px;
}

.tagline {
  font-size: 0.85rem;
  color: var(--text-secondary);
}

/* Main */
.app-main {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.hero {
  text-align: center;
}

.hero h2 {
  font-size: 1.6rem;
  margin-bottom: 6px;
}

.hero p {
  color: var(--text-secondary);
  font-size: 0.95rem;
}

/* Historique */
.history {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  list-style: none;
  justify-content: center;
}

.history-btn {
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid var(--card-border);
  color: var(--text-primary);
  padding: 6px 12px;
  border-radius: 999px;
  font-size: 0.85rem;
  cursor: pointer;
  transition: background 0.2s ease, transform 0.2s ease;
}

.history-btn:hover {
  background: rgba(255, 255, 255, 0.28);
  transform: translateY(-2px);
}

/* Erreur */
.error-box {
  background: var(--error-bg);
  border: 1px solid var(--error-border);
  color: #ffe0e0;
  padding: 14px 18px;
  border-radius: var(--radius);
  font-size: 0.95rem;
  animation: fadeIn 0.35s ease;
}

/* Empty state */
.empty-state {
  text-align: center;
  padding: 40px 20px;
  background: var(--card-bg);
  border: 1px dashed var(--card-border);
  border-radius: var(--radius);
  color: var(--text-secondary);
}

/* Status line (v-show) */
.status-line {
  text-align: center;
  font-size: 0.8rem;
  color: var(--text-secondary);
  font-style: italic;
}

/* Footer */
.app-footer {
  text-align: center;
  font-size: 0.8rem;
  color: var(--text-secondary);
  padding: 10px;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-6px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* Responsive */
@media (max-width: 520px) {
  .app-header h1 { font-size: 1.2rem; }
  .hero h2 { font-size: 1.3rem; }
  .logo { font-size: 1.9rem; }
}
</style>