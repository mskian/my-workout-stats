<script setup>
import { ref, onMounted, computed } from "vue";
import { BarChart3, Clock, TrendingUp } from "lucide-vue-next";

const dataUrl =
  "https://santhoshkumar.github.io/steps-data/data/live.csv";

const stats = ref(null);
const hourlySteps = ref({});


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

onMounted(async () => {
  try {
    const res = await fetch(dataUrl, { cache: "no-store" });
    const csv = await res.text();

    const parsed = parseCSV(csv);

    if (!parsed.length) {
      console.error("No valid CSV rows parsed");
      return;
    }

    const result = calculate(parsed);
    stats.value = result;
    hourlySteps.value = result.hourly;
  } catch (err) {
    console.error("CSV load failed:", err);
  }
});
</script>

<template>
<section class="section">
  <div class="container" v-if="stats">

    <div class="columns is-multiline mb-5">

      <div class="column is-4">
        <div class="m-card" :style="{ background: 'var(--distance-bg)' }">
          <div class="m-icon icon-distance">
            <TrendingUp :size="26" />
          </div>
          <div class="m-title">Total Steps</div>
          <div class="m-value">{{ stats.total.toLocaleString() }}</div>
        </div>
      </div>

      <div class="column is-4">
        <div class="m-card" :style="{ background: 'var(--activity-bg)' }">
          <div class="m-icon icon-activity">
            <Clock :size="26" />
          </div>
          <div class="m-title">Avg / Hour</div>
          <div class="m-value">{{ stats.perHour.toLocaleString() }}</div>
        </div>
      </div>

      <div class="column is-4">
        <div class="m-card" :style="{ background: 'var(--time-bg)' }">
          <div class="m-icon icon-time">
            <BarChart3 :size="26" />
          </div>
          <div class="m-title">Max Hour</div>
          <div class="m-value">{{ stats.maxHour.toLocaleString() }}</div>
        </div>
      </div>

    </div>

    <div class="hourly-list">

      <div v-for="h in hours" :key="h" class="hour-row">
        <div class="hour-label">{{ hourLabel(h) }}</div>

        <div class="hour-bar-wrap">
          <div
            class="hour-bar"
            :style="{
              width: ((hourlySteps[h] || 0) / maxHourly * 100) + '%'
            }"
          ></div>
        </div>

        <div class="hour-value">
          {{ (hourlySteps[h] || 0).toLocaleString() }}
        </div>
      </div>

    </div>

  </div>
</section>
</template>