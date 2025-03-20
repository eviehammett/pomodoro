<template>
  <div :class="theme" class="min-h-screen relative">
    <PomodoroTimer />
  </div>
</template>

<script>
import { ref, provide, watch, onMounted } from 'vue';
import PomodoroTimer from './components/PomodoroTimer.vue';

export default {
  components: { PomodoroTimer },
  setup() {
    const theme = ref(localStorage.getItem('theme') || 'theme-dark');

    // Ensure the theme updates instantly
    const applyTheme = (newTheme) => {
      document.body.classList.remove('theme-dark', 'theme-light', 'theme-cozy');
      document.body.classList.add(newTheme);
      localStorage.setItem('theme', newTheme);
    };

    watch(theme, (newTheme) => {
      applyTheme(newTheme);
    });

    onMounted(() => {
      applyTheme(theme.value);
    });

    provide('theme', theme); // Provide the theme for child components

    return { theme };
  },
};
</script>


<style>
/* Global theme styles */
.theme-dark {
  background-color: #2d3748; /* bg-gray-800 */
  color: #ffffff; /* text-white */
}

.theme-light {
  background-color: #f7fafc; /* bg-gray-100 */
  color: #1a202c; /* text-gray-900 */
}

.theme-cozy {
  background-color: #fffaf0; /* bg-yellow-100 */
  color: #5a3e36; /* text-brown-900 */
}
</style>