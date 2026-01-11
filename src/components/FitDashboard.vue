<script setup>
import { ref, onMounted, computed } from "vue";
import {
  Activity,
  Flame,
  HeartPulse,
  UserCheck,
  BadgeCheck,
} from "lucide-vue-next";

const API_URL =
  "https://mskian.github.io/strava-running-stats/fit.json";

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
      headers: { Accept: "application/json" },
    });

    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    if (retries > 0) return fetchWithRetry(url, retries - 1);
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
    if (!data?.metrics || !data?.training_status) {
      throw new Error("Invalid API structure");
    }
    stats.value = data;
  } catch (err) {
    error.value = "Failed to load training status. Please retry.";
  } finally {
    isLoading.value = false;
  }
}

onMounted(load);

const formClass = computed(() => {
  const v = stats.value?.metrics?.form_tsb ?? 0;
  if (v > 10) return "form-good";
  if (v < -10) return "form-bad";
  return "form-neutral";
});

const trainingBadgeStyle = computed(() => ({
  background: stats.value?.training_status?.color || "#9ca3af",
}));
</script>

<template>
  <section class="hero-section" v-if="stats">
    <h1 class="hero-title">{{ stats.body_status }}</h1>
    <p class="hero-sub">{{ stats.recommendation }}</p>
  </section>
  <section v-else-if="isLoading" class="section">
    <div class="container has-text-centered">
      Loading training status…
    </div>
  </section>
  <section v-else-if="error" class="section">
    <div class="container has-text-centered has-text-danger">
      {{ error }}
    </div>
  </section>
  <section v-if="stats" class="section">
    <div class="container">
      <div class="columns is-multiline is-align-stretch">
        <div class="column is-3 is-flex">
          <div class="m-card m-card-fill card-fitness">
            <div class="m-icon icon-green">
              <HeartPulse :size="26" />
            </div>
            <div class="m-title">Fitness (CTL)</div>
            <div class="m-value">{{ stats.metrics.fitness_ctl }}</div>
          </div>
        </div>
        <div class="column is-3 is-flex">
          <div class="m-card m-card-fill card-fatigue">
            <div class="m-icon icon-orange">
              <Flame :size="26" />
            </div>
            <div class="m-title">Fatigue (ATL)</div>
            <div class="m-value">{{ stats.metrics.fatigue_atl }}</div>
          </div>
        </div>
        <div class="column is-3 is-flex">
          <div class="m-card m-card-fill form-card" :class="formClass">
            <div class="m-icon icon-form">
              <Activity :size="26" />
            </div>
            <div class="m-title">Form (TSB)</div>
            <div class="m-value">{{ stats.metrics.form_tsb }}</div>
            <div class="m-sub">
              {{
                formClass === "form-good"
                  ? "Fresh"
                  : formClass === "form-bad"
                  ? "Fatigued"
                  : "Balanced"
              }}
            </div>
          </div>
        </div>
        <div class="column is-3 is-flex">
          <div class="m-card m-card-fill card-body">
            <div class="m-icon icon-purple">
              <UserCheck :size="26" />
            </div>
            <div class="m-title">Body Status</div>
            <div class="m-value body-text">
              {{ stats.body_status }}
            </div>
            <div class="training-badge" :style="trainingBadgeStyle">
              <BadgeCheck :size="14" />
              {{ stats.training_status.label }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
