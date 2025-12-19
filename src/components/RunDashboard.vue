<script setup>
import { ref, onMounted } from "vue";
import { Route, Activity, Timer } from "lucide-vue-next";

const API_URL =
  "https://mskian.github.io/strava-running-stats/data.json";

const period = ref("week");
const stats = ref(null);
const isLoading = ref(true);
const error = ref(null);

async function fetchWithRetry(url, retries = 1) {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), 8000);

  try {
    const res = await fetch(`${url}?v=${Date.now()}`, {
      signal: controller.signal,
      cache: "no-store",
      headers: {
        Accept: "application/json",
      },
    });

    if (!res.ok) {
      throw new Error(`HTTP ${res.status}`);
    }

    return await res.json();
  } catch (err) {
    if (retries > 0) {
      return fetchWithRetry(url, retries - 1);
    }
    throw err;
  } finally {
    clearTimeout(timeout);
  }
}

async function load() {
  isLoading.value = true;
  error.value = null;

  try {
    const data = await fetchWithRetry(API_URL, 1);

    if (!data || !data.week || !data.month || !data.year) {
      throw new Error("Invalid API structure");
    }

    stats.value = data;
  } catch (err) {
    console.error("Stats load failed:", err);
    error.value = "Failed to load running data. Please retry.";
  } finally {
    isLoading.value = false;
  }
}

onMounted(load);
</script>

<template>
<section class="section">
  <div class="container">

    <div class="period-toggle-wrap">
      <div class="period-toggle" role="tablist" aria-label="Time period">
        <button
          v-for="p in ['week','month','year']"
          :key="p"
          class="period-btn"
          :class="{ active: period === p }"
          @click="period = p"
        >
          {{ p.charAt(0).toUpperCase() + p.slice(1) }}
        </button>
      </div>
    </div>

    <div v-if="isLoading" class="columns is-multiline">
      <div class="column is-4" v-for="i in 3" :key="i">
        <div class="m-card skeleton"></div>
      </div>
    </div>

    <div v-else-if="error" class="notification is-danger is-light">
      {{ error }}
      <br />
      <button class="button is-small mt-2" @click="load">
        Retry
      </button>
    </div>

    <div v-else class="columns is-multiline">

      <div class="column is-4">
        <div class="m-card" :style="{ background: 'var(--distance-bg)' }">
          <div class="m-icon icon-distance">
            <Route :size="26" />
          </div>
          <div class="m-title">Distance</div>
          <div class="m-value">
            {{ stats[period].totalDistanceKm }} km
          </div>
        </div>
      </div>

      <div class="column is-4">
        <div class="m-card" :style="{ background: 'var(--activity-bg)' }">
          <div class="m-icon icon-activity">
            <Activity :size="26" />
          </div>
          <div class="m-title">Activities</div>
          <div class="m-value">
            {{ stats[period].totalActivities }}
          </div>
        </div>
      </div>

      <div class="column is-4">
        <div class="m-card" :style="{ background: 'var(--time-bg)' }">
          <div class="m-icon icon-time">
            <Timer :size="26" />
          </div>
          <div class="m-title">Total Time</div>
          <div class="m-value">
            {{ stats[period].totalTime }}
          </div>
        </div>
      </div>

    </div>

  </div>
</section>
</template>

<style scoped>
.skeleton {
  height: 170px;
  border-radius: 22px;
  background: linear-gradient(
    100deg,
    rgba(255,255,255,0.3) 40%,
    rgba(255,255,255,0.6) 50%,
    rgba(255,255,255,0.3) 60%
  );
  background-size: 200% 100%;
  animation: shimmer 1.2s infinite linear;
}

@keyframes shimmer {
  to {
    background-position-x: -200%;
  }
}
</style>
