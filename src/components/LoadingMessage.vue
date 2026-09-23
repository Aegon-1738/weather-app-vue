<template>
  <div
    class="loading-container fade-in-up"
    role="status"
    aria-live="polite"
    aria-label="Loading weather data"
  >
    <div class="spinner" aria-hidden="true"></div>
    <p class="loading-text">Fetching weather data...</p>
  </div>
</template>

<script>
export default {
  name: 'LoadingMessage',
}
</script>

<style scoped>
.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  padding: 3rem 1.5rem;
  background: var(--color-card);
  border: 1px solid var(--color-card-border);
  border-radius: var(--radius);
  backdrop-filter: blur(12px);
}

.spinner {
  width: 48px;
  height: 48px;
  border: 4px solid rgba(255, 255, 255, 0.15);
  border-top-color: var(--color-primary);
  border-radius: 50%;
  animation: spin 0.9s linear infinite;
}

.loading-text {
  color: var(--color-text-muted);
  font-size: 0.95rem;
  font-weight: 500;
  letter-spacing: 0.02em;
}
</style>