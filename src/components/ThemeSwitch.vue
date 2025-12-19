<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import { Sun, Moon } from "lucide-vue-next";

const STORAGE_KEY = "ui-theme";
const dark = ref(false);

function applyTheme(theme) {
  document.documentElement.setAttribute("data-theme", theme);
  localStorage.setItem(STORAGE_KEY, theme);
  dark.value = theme === "dark";
}

function getSystemTheme() {
  return window.matchMedia("(prefers-color-scheme: dark)").matches
    ? "dark"
    : "light";
}

function loadTheme() {
  const saved = localStorage.getItem(STORAGE_KEY);
  applyTheme(saved || getSystemTheme());
}

function toggleTheme() {
  applyTheme(dark.value ? "light" : "dark");
}

function handleStorageChange(e) {
  if (e.key === STORAGE_KEY && e.newValue) {
    applyTheme(e.newValue);
  }
}

function handleVisibility() {
  loadTheme();
}

onMounted(() => {
  loadTheme();

  window.addEventListener("storage", handleStorageChange);
  window.addEventListener("visibilitychange", handleVisibility);
});

onBeforeUnmount(() => {
  window.removeEventListener("storage", handleStorageChange);
  window.removeEventListener("visibilitychange", handleVisibility);
});
</script>

<template>
  <button
    class="theme-btn"
    type="button"
    aria-label="Toggle theme"
    @click="toggleTheme"
  >
    <Sun v-if="!dark" :size="20" />
    <Moon v-else :size="20" />
  </button>
</template>
