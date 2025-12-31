<script setup>
import {
  Bike,
  CalendarDays,
  Route,
  Clock,
  Mountain,
  BadgeCheck,
} from "lucide-vue-next";

const stats = [
  { label: "Total Days Active", value: "23", icon: CalendarDays },
  { label: "Total Distance", value: "640 km", icon: Route },
  { label: "Total Time", value: "31h 29m", icon: Clock },
  { label: "Total Elevation", value: "1,838 m", icon: Mountain },
];
</script>

<template>
  <div class="is-flex is-justify-content-center px-3 mt-5 mb-6">
    <div class="card-amoled p-5" style="max-width:420px;width:100%">
        <p class="stat-label mb-4">Back in the saddle after a long gap — my last ride was on November 7, 2024, and I started again on November 19, 2025 with a new cycle.</p>
      <div class="is-flex is-align-items-center mb-5">
        <div>
          <div class="is-flex is-align-items-center mt-2">
            <Bike :size="36" class="accent-icon" />
            <h2 class="title is-4 ml-2">Cycling</h2>
          </div>
        </div>
        <div class="ml-auto stat-badge is-flex is-align-items-center is-justify-content-center">
          <BadgeCheck :size="28" />
        </div>
      </div>
      <div v-for="item in stats" :key="item.label" class="mb-4">
        <p class="stat-label">{{ item.label }}</p>
        <div class="is-flex is-align-items-center is-justify-content-space-between">
          <p class="stat-value">{{ item.value }}</p>
          <component :is="item.icon" :size="22" class="accent-icon" />
        </div>
      </div>
      <div class="year-strip mt-5">
        STRAVA YEAR IN SPORT 2025
      </div>
    </div>
</div>
</template>

<style scoped>
.card-amoled {
  animation: rise 0.45s ease-out;
  will-change: transform, opacity;
}

@keyframes rise {
  from {
    opacity: 0;
    transform: translateY(12px);
  }
  to {
    opacity: 1;
    transform: none;
  }
}
</style>
