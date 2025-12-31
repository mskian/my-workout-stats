<script setup>
import { ref, onMounted, watch } from "vue";
import { Sun, Moon } from "lucide-vue-next";

const ORANGE_AMOLED = ["#ff6d00", "#ff7a18", "#ff8f00", "#ff9100"];
const GREEN_LIGHT = ["#00c853", "#00e676", "#2e7d32", "#43a047"];

const dark = ref(true);

const setAccent = (theme) => {
  const palette = theme === "dark" ? ORANGE_AMOLED : GREEN_LIGHT;
  const color = palette[Math.floor(Math.random() * palette.length)];
  document.documentElement.style.setProperty("--accent", color);
};

const applyTheme = () => {
  const theme = dark.value ? "dark" : "light";
  document.documentElement.setAttribute("data-theme", theme);
  localStorage.setItem("stats-theme", theme);
  setAccent(theme);
};

onMounted(() => {
  const saved = localStorage.getItem("stats-theme");
  dark.value = saved !== "light";
  applyTheme();
});

watch(dark, applyTheme);
</script>

<template>
  <div class="theme-pill">
    <button @click="dark = false" aria-label="Light">
      <Sun :size="16" />
    </button>
    <button @click="dark = true" aria-label="Dark">
      <Moon :size="16" />
    </button>
  </div>
</template>

<style scoped>
button {
  border: none;
  background: transparent;
  padding: 0.45rem 0.6rem;
  border-radius: 999px;
  color: var(--text);
  cursor: pointer;
}
button:hover {
  background: color-mix(in srgb, var(--accent), transparent 85%);
}

button:active {
  transform: scale(0.96);
}
</style>
