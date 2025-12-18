<script setup>
import { ref, onMounted } from "vue";
import { Route, Activity, Timer } from "lucide-vue-next";

const period = ref("week");
const stats = ref(null);

async function load() {
  const res = await fetch(
    "https://mskian.github.io/strava-running-stats/data.json"
  );
  stats.value = await res.json();
}

onMounted(load);
</script>

<template>
<section class="section">
  <div class="container">
    <div class="period-toggle-wrap">
      <div class="period-toggle" role="tablist" aria-label="Time period">
        <button
          class="period-btn"
          :class="{ active: period === 'week' }"
          @click="period = 'week'"
          role="tab"
        >
          Week
        </button>

        <button
          class="period-btn"
          :class="{ active: period === 'month' }"
          @click="period = 'month'"
          role="tab"
        >
          Month
        </button>

        <button
          class="period-btn"
          :class="{ active: period === 'year' }"
          @click="period = 'year'"
          role="tab"
        >
          Year
        </button>
      </div>
    </div>

    <div class="columns is-multiline" v-if="stats">

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