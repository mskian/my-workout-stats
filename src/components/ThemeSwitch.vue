<script setup>
import { ref, onMounted } from "vue";
import { Sun, Moon } from "lucide-vue-next";

const dark = ref(false);

const applyTheme = () => {
  const theme = dark.value ? "dark" : "light";
  document.documentElement.setAttribute("data-theme", theme);
  localStorage.setItem("ui-theme", theme);
};

const toggleTheme = () => {
  dark.value = !dark.value;
  applyTheme();
};

onMounted(() => {
  dark.value = localStorage.getItem("ui-theme") === "dark";
  applyTheme();
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
