<!-- src/components/SearchBar.vue -->
<template>
  <form class="search-bar" @submit.prevent="onSubmit">
    <!-- v-model sur l'input (liaison bidirectionnelle) -->
    <label for="city-input" class="sr-only">City name</label>
    <input
      id="city-input"
      type="text"
      v-model="city"
      placeholder="Enter city (e.g. Bujumbura, Tokyo...)"
      autocomplete="off"
      :disabled="disabled"
      aria-label="City name"
    />

    <!-- v-bind (:) sur disabled et aria-busy -->
    <button
      type="submit"
      class="search-btn"
      :disabled="disabled"
      :aria-busy="disabled"
    >
      {{ disabled ? 'Searching...' : 'Search' }}
    </button>
  </form>
</template>

<script>
export default {
  name: 'SearchBar',

  props: {
    // Permet au parent de désactiver la barre pendant le chargement
    disabled: {
      type: Boolean,
      default: false
    }
  },

  // Le composant gère sa propre saisie localement
  data() {
    return {
      city: ''
    }
  },

  methods: {
    onSubmit() {
      // Émet l'événement vers App.vue avec la valeur saisie
      this.$emit('search', this.city)
      // On vide le champ après envoi (optionnel mais propre)
      this.city = ''
    }
  }
}
</script>

<style scoped>
.search-bar {
  display: flex;
  gap: 10px;
  background: var(--card-bg);
  border: 1px solid var(--card-border);
  padding: 10px;
  border-radius: var(--radius);
  backdrop-filter: blur(12px);
  box-shadow: var(--shadow);
}

.search-bar input {
  flex: 1;
  padding: 14px 16px;
  border-radius: 12px;
  border: 1px solid transparent;
  background: rgba(255, 255, 255, 0.9);
  color: #1e3c72;
  font-size: 1rem;
  outline: none;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

.search-bar input:focus {
  border-color: var(--accent);
  box-shadow: 0 0 0 3px rgba(255, 217, 61, 0.35);
}

.search-bar input:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.search-btn {
  padding: 14px 24px;
  border: none;
  border-radius: 12px;
  background: var(--accent);
  color: #1e3c72;
  font-weight: 700;
  font-size: 1rem;
  cursor: pointer;
  transition: transform 0.15s ease, box-shadow 0.2s ease, background 0.2s ease;
}

.search-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 18px rgba(255, 217, 61, 0.4);
}

.search-btn:active:not(:disabled) {
  transform: translateY(0);
}

.search-btn:disabled {
  background: #cfcfcf;
  color: #666;
  cursor: not-allowed;
}

/* Classe utilitaire pour lecteurs d'écran */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

/* Responsive : empile sur mobile */
@media (max-width: 520px) {
  .search-bar {
    flex-direction: column;
  }
  .search-btn {
    width: 100%;
  }
}
</style>