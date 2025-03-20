<template>
  <div class="relative">
    <button @click="toggleMenu" class="p-2 bg-gray-700 text-white rounded">
      &#9776;
    </button>
    <div v-if="menuOpen" class="absolute right-0 mt-2 w-40 bg-white shadow-lg rounded">
      <button 
        v-for="themeOption in themes" 
        :key="themeOption.value" 
        @click="setTheme(themeOption.value)" 
        :disabled="theme === themeOption.value"
        class="block w-full text-left px-4 py-2 hover:bg-gray-200"
        :class="{ 'bg-gray-200': theme === themeOption.value }"
      >
        {{ themeOption.label }}
      </button>
    </div>
  </div>
</template>

<script>
import { ref, inject } from 'vue';

export default {
  setup(_, { emit }) {
    const menuOpen = ref(false);
    const themes = [
      { label: 'Dark Mode', value: 'theme-dark' },
      { label: 'Light Mode', value: 'theme-light' },
      { label: 'Cozy Mode', value: 'theme-cozy' },
    ];
    const theme = inject('theme'); // Get the reactive theme

    const toggleMenu = () => {
      menuOpen.value = !menuOpen.value;
    };

    const setTheme = (newTheme) => {
      theme.value = newTheme; // This now updates globally
      localStorage.setItem('theme', newTheme);
      emit('theme-changed', newTheme);
      menuOpen.value = false;
    };

    return {
      menuOpen,
      themes,
      theme,
      toggleMenu,
      setTheme,
    };
  },
};
</script>
