<!-- src/components/WeatherCard.vue -->
<template>
  <article class="weather-card">
    <!-- En-tête : ville + pays -->
    <header class="card-header">
      <h2 class="city">{{ weather.city }}</h2>
      <!-- v-if : affiche le pays uniquement s'il est présent -->
      <p v-if="weather.country" class="country">{{ weather.country }}</p>
    </header>

    <!-- Bloc principal : icône + température -->
    <div class="card-body">
      <!-- v-bind (:) sur aria-label pour l'accessibilité -->
      <div class="weather-icon" :aria-label="weather.condition" role="img">
        {{ weather.icon }}
      </div>
      <div class="temperature">
        {{ weather.temperature }}<span class="unit">°C</span>
      </div>
      <p class="condition">{{ weather.condition }}</p>
    </div>

    <!-- Détails : humidité + vent -->
    <ul class="details">
      <li>
        <span class="label">💧 Humidity</span>
        <span class="value">{{ weather.humidity }}%</span>
      </li>
      <li>
        <span class="label">💨 Wind</span>
        <span class="value">{{ weather.windSpeed }} km/h</span>
      </li>
    </ul>
  </article>
</template>

<script>
export default {
  name: 'WeatherCard',

  // Prop obligatoire contenant les données formatées par App.vue
  props: {
    weather: {
      type: Object,
      required: true
    }
  }
}
</script>

<style scoped>
.weather-card {
  background: var(--card-bg);
  border: 1px solid var(--card-border);
  border-radius: var(--radius);
  padding: 26px 24px;
  backdrop-filter: blur(14px);
  box-shadow: var(--shadow);
  animation: cardAppear 0.45s ease-out;
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.card-header {
  text-align: center;
}

.city {
  font-size: 1.7rem;
  letter-spacing: 0.5px;
}

.country {
  font-size: 0.9rem;
  color: var(--text-secondary);
  margin-top: 2px;
}

.card-body {
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
}

.weather-icon {
  font-size: 4rem;
  line-height: 1;
  animation: float 3s ease-in-out infinite;
}

.temperature {
  font-size: 3.4rem;
  font-weight: 700;
  letter-spacing: -2px;
}

.temperature .unit {
  font-size: 1.6rem;
  font-weight: 400;
  margin-left: 4px;
  color: var(--text-secondary);
}

.condition {
  font-size: 1.05rem;
  color: var(--text-secondary);
  text-transform: capitalize;
}

.details {
  list-style: none;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
  margin-top: 6px;
}

.details li {
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid var(--card-border);
  border-radius: 12px;
  padding: 14px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
}

.details .label {
  font-size: 0.85rem;
  color: var(--text-secondary);
}

.details .value {
  font-size: 1.25rem;
  font-weight: 600;
}

@keyframes cardAppear {
  from { opacity: 0; transform: translateY(14px) scale(0.98); }
  to   { opacity: 1; transform: translateY(0) scale(1); }
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-6px); }
}

/* Responsive */
@media (max-width: 520px) {
  .temperature { font-size: 2.8rem; }
  .weather-icon { font-size: 3.2rem; }
  .details { grid-template-columns: 1fr; }
}
</style>