<script setup>
import { ref, onMounted, computed } from "vue";
import { BarChart3, Clock, TrendingUp } from "lucide-vue-next";

const DATA_URL =
  "https://santhoshkumar.github.io/steps-data/data/live.csv";

const stats = ref(null);
const hourlySteps = ref({});
const isLoading = ref(true);
const error = ref(null);

async function fetchWithRetry(url, retries = 1) {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), 8000);

  try {
    const res = await fetch(`${url}?v=${Date.now()}`, {
      signal: controller.signal,
      cache: "no-store",
      headers: { Accept: "text/csv" },
    });

    if (!res.ok) {
      throw new Error(`HTTP ${res.status}`);
    }

    return await res.text();
  } catch (err) {
    if (retries > 0) {
      return fetchWithRetry(url, retries - 1);
    }
    throw err;
  } finally {
    clearTimeout(timeout);
  }
}

function parseCSV(text) {
  const lines = text.trim().split("\n");
  if (lines.length <= 1) return [];

  return lines.slice(1).map(line => {
    const [dateStr, _timeStr, stepsStr] = line.split(",");
    if (!dateStr || !stepsStr) return null;

    const [datePart, timePart] = dateStr.trim().split(" ");
    if (!datePart || !timePart) return null;

    const [year, month, day] = datePart.split(".").map(Number);
    const [hour, min, sec] = timePart.split(":").map(Number);

    const date = new Date(year, month - 1, day, hour, min, sec);
    const steps = Number(stepsStr);

    if (isNaN(date.getTime()) || isNaN(steps)) return null;

    return { date, steps };
  }).filter(Boolean);
}


function groupByHour(data) {
  return data.reduce((acc, d) => {
    const h = d.date.getHours();
    acc[h] = (acc[h] || 0) + d.steps;
    return acc;
  }, {});
}

function calculate(data) {
  const hourly = groupByHour(data);
  const values = Object.values(hourly);

  const total = values.reduce((a, b) => a + b, 0);
  const hoursElapsed = Math.max(1, new Date().getHours());
  const minsElapsed = Math.max(1, hoursElapsed * 60);

  return {
    total,
    perHour: Math.round(total / hoursElapsed),
    perMin: Math.round(total / minsElapsed),
    maxHour: values.length ? Math.max(...values) : 0,
    hourly
  };
}


const hours = Array.from({ length: 24 }, (_, i) => i);

const maxHourly = computed(() =>
  Math.max(...Object.values(hourlySteps.value), 1)
);

function hourLabel(h) {
  if (h === 0) return "12 AM";
  if (h === 12) return "12 PM";
  return h < 12 ? `${h} AM` : `${h - 12} PM`;
}

async function load() {
  isLoading.value = true;
  error.value = null;

  try {
    const csv = await fetchWithRetry(DATA_URL, 1);
    const parsed = parseCSV(csv);

    if (!parsed.length) {
      throw new Error("No valid CSV rows parsed");
    }

    const result = calculate(parsed);
    stats.value = result;
    hourlySteps.value = result.hourly;
  } catch (err) {
    console.error("Live steps load failed:", err);
    error.value = "Failed to load live steps data.";
  } finally {
    isLoading.value = false;
  }
}

onMounted(load);
</script>

<template>
<section class="section">
  <div class="container">

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

    <div v-else>

      <div class="columns is-multiline mb-5">

        <div class="column is-4">
          <div class="m-card" :style="{ background: 'var(--distance-bg)' }">
            <div class="m-icon icon-distance">
              <TrendingUp :size="26" />
            </div>
            <div class="m-title">Total Steps</div>
            <div class="m-value">
              {{ stats.total.toLocaleString() }}
            </div>
          </div>
        </div>

        <div class="column is-4">
          <div class="m-card" :style="{ background: 'var(--activity-bg)' }">
            <div class="m-icon icon-activity">
              <Clock :size="26" />
            </div>
            <div class="m-title">Avg / Hour</div>
            <div class="m-value">
              {{ stats.perHour.toLocaleString() }}
            </div>
          </div>
        </div>

        <div class="column is-4">
          <div class="m-card" :style="{ background: 'var(--time-bg)' }">
            <div class="m-icon icon-time">
              <BarChart3 :size="26" />
            </div>
            <div class="m-title">Max Hour</div>
            <div class="m-value">
              {{ stats.maxHour.toLocaleString() }}
            </div>
          </div>
        </div>

      </div>

      <div class="hourly-list">

        <div v-for="h in hours" :key="h" class="hour-row">
          <div class="hour-label">{{ hourLabel(h) }}</div>

          <div class="hour-bar-wrap">
            <div
              class="hour-bar"
              :style="{ width: ((hourlySteps[h] || 0) / maxHourly * 100) + '%' }"
            ></div>
          </div>

          <div class="hour-value">
            {{ (hourlySteps[h] || 0).toLocaleString() }}
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
