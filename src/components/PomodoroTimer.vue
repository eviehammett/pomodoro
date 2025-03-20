<template>
  <div :class="['max-w-md mx-auto p-6 rounded-lg shadow-lg text-center relative', themeClass]">
    <ThemeSwitcher @theme-changed="updateTheme" class="fixed top-4 right-4 z-50" />
    
    <h1 class="text-3xl font-bold mb-4">{{ mode === 'focus' ? 'Focus Time' : 'Break Time' }}</h1>
    <p class="text-6xl font-mono mb-6">{{ formattedTime }}</p>
    
    <div class="space-x-4 mb-4">
      <button @click="switchTimer" class="px-4 py-2 bg-blue-500 hover:bg-blue-600 rounded">Switch Timer</button>
    </div>

    <div class="space-x-4">
      <button @click="startTimer" :disabled="isRunning" class="px-4 py-2 bg-green-500 hover:bg-green-600 rounded">Start</button>
      <button @click="pauseTimer" :disabled="!isRunning" class="px-4 py-2 bg-yellow-500 hover:bg-yellow-600 rounded">Pause</button>
      <button @click="resetTimer" class="px-4 py-2 bg-red-500 hover:bg-red-600 rounded">Reset</button>
    </div>
    
    <p class="mt-4 text-sm">Completed Sessions: {{ sessionCount }}</p>
    <p class="mt-2 text-sm">All Time Sessions Completed: {{ allTimeSessions }}</p>
  </div>
</template>

<script>
import { defineComponent, ref, computed, watch, onMounted } from 'vue';
import { Howl } from 'howler';
import ThemeSwitcher from './ThemeSwitcher.vue';

export default defineComponent({
  components: { ThemeSwitcher },
  setup() {
    const focusDuration = ref(parseInt(localStorage.getItem('focusDuration')) || 25 * 60);
    const breakDuration = ref(focusDuration.value === 25 * 60 ? 5 * 60 : 10 * 60);
    const altFocusDuration = ref(50 * 60);
    const altBreakDuration = ref(10 * 60);
    const timeLeft = ref(focusDuration.value);
    const isRunning = ref(false);
    const sessionCount = ref(0);
    const allTimeSessions = ref(parseInt(localStorage.getItem('allTimeSessions')) || 0);
    const mode = ref('focus');
    const themes = ['theme-dark', 'theme-light', 'theme-cozy'];
    const theme = ref(localStorage.getItem('theme') || 'theme-dark');
    let timer;

    const alarm = new Howl({ 
        src: [new URL('../assets/alarm.wav', import.meta.url).href], 
        html5: true,
        volume: 1.0 
    });

    const themeClass = computed(() => {
      console.log('Current theme:', theme.value); // Debugging line
      return {
        'theme-dark': 'bg-gray-800 text-white',
        'theme-light': 'bg-gray-100 text-gray-900',
        'theme-cozy': 'bg-yellow-100 text-brown-900',
      }[theme.value];
    });

    const formattedTime = computed(() => {
      const minutes = Math.floor(timeLeft.value / 60);
      const seconds = timeLeft.value % 60;
      return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
    });

    const startTimer = () => {
      if (!isRunning.value) {
        isRunning.value = true;
        timer = setInterval(() => {
          if (timeLeft.value > 0) {
            timeLeft.value--;
          } else {
            alarm.play();
            switchMode();
          }
        }, 1000);
      }
    };

    const pauseTimer = () => {
      clearInterval(timer);
      isRunning.value = false;
    };

    const resetTimer = () => {
      pauseTimer();
      timeLeft.value = mode.value === 'focus' ? focusDuration.value : breakDuration.value;
    };

    const switchMode = () => {
      if (mode.value === 'focus') {
        mode.value = 'break';
        timeLeft.value = breakDuration.value;
      } else {
        mode.value = 'focus';
        timeLeft.value = focusDuration.value;
        sessionCount.value++;
        allTimeSessions.value++;
        localStorage.setItem('allTimeSessions', allTimeSessions.value);
      }
    };

    const switchTimer = () => {
      pauseTimer();
      if (focusDuration.value === 25 * 60) {
        focusDuration.value = altFocusDuration.value;
        breakDuration.value = altBreakDuration.value;
      } else {
        focusDuration.value = 25 * 60;
        breakDuration.value = 5 * 60;
      }
      localStorage.setItem('focusDuration', focusDuration.value);
      localStorage.setItem('breakDuration', breakDuration.value);
      mode.value = 'focus';
      timeLeft.value = focusDuration.value;
    };

    const updateTheme = (newTheme) => {
      console.log('Updating theme to:', newTheme); // Debugging line
      theme.value = newTheme;
      localStorage.setItem('theme', newTheme);
    };

    onMounted(() => {
      allTimeSessions.value = parseInt(localStorage.getItem('allTimeSessions')) || 0;
    });

    return {
      timeLeft,
      formattedTime,
      isRunning,
      sessionCount,
      allTimeSessions,
      startTimer,
      pauseTimer,
      resetTimer,
      mode,
      switchTimer,
      themeClass,
      updateTheme,
    };
  },
});
</script>

<style scoped>
button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.themeClass {
  width: 100%;
  height: 100%;
}
</style>