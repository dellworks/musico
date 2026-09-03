<script setup lang="ts">
import { onMounted, onUnmounted, ref } from "vue";
import { useThemeStore } from "../stores/theme";

const theme = useThemeStore();
const open = ref(false);
const root = ref<HTMLElement | null>(null);

function onDocClick(event: MouseEvent) {
  if (!root.value?.contains(event.target as Node)) {
    open.value = false;
  }
}

function choose(dark: boolean) {
  theme.setDark(dark);
}

onMounted(() => document.addEventListener("click", onDocClick));
onUnmounted(() => document.removeEventListener("click", onDocClick));
</script>

<template>
  <div ref="root" class="relative">
    <button
      type="button"
      class="grid h-11 w-11 place-items-center rounded-full text-zinc-500 ring-1 ring-zinc-300 hover:text-zinc-900 dark:text-zinc-400 dark:ring-white/15 dark:hover:text-white"
      :aria-expanded="open"
      aria-label="配置"
      @click="open = !open"
    >
      <svg viewBox="0 0 24 24" class="h-5 w-5" fill="currentColor" aria-hidden="true">
        <rect x="4" y="6" width="16" height="2.2" rx="1.1" />
        <rect x="4" y="11" width="16" height="2.2" rx="1.1" />
        <rect x="4" y="16" width="16" height="2.2" rx="1.1" />
      </svg>
    </button>
    <div
      v-if="open"
      class="absolute right-0 top-full z-30 mt-1 w-44 overflow-hidden rounded-2xl bg-white py-1 shadow-xl ring-1 ring-zinc-200 dark:bg-zinc-900 dark:ring-white/10"
    >
      <p class="px-3 py-1 text-xs text-zinc-400">外观</p>
      <button
        type="button"
        class="flex min-h-11 w-full items-center px-3 text-left text-sm"
        :class="
          !theme.dark
            ? 'bg-zinc-900 text-white dark:bg-white dark:text-zinc-900'
            : 'hover:bg-zinc-100 dark:hover:bg-white/10'
        "
        @click="choose(false)"
      >
        浅色
      </button>
      <button
        type="button"
        class="flex min-h-11 w-full items-center px-3 text-left text-sm"
        :class="
          theme.dark
            ? 'bg-zinc-900 text-white dark:bg-white dark:text-zinc-900'
            : 'hover:bg-zinc-100 dark:hover:bg-white/10'
        "
        @click="choose(true)"
      >
        深色
      </button>
    </div>
  </div>
</template>
