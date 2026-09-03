<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from "vue";
import type { CatalogChart, CatalogGroup } from "../types";

const props = defineProps<{
  name: string;
  chartKey: string;
  groups: CatalogGroup[];
}>();

const emit = defineEmits<{
  select: [key: string];
  move: [key: string, direction: "up" | "down"];
}>();

const open = ref(false);
const query = ref("");
const root = ref<HTMLElement | null>(null);

function isSongChart(chart: CatalogChart): boolean {
  if (chart.playable === false) return false;
  const name = chart.name.toUpperCase();
  return !name.includes("MV") && !name.includes("视频榜") && !name.includes("专辑榜") && !name.includes("歌手榜");
}

const ordered = computed(() => {
  const items: CatalogChart[] = [];
  for (const group of props.groups) {
    for (const chart of group.charts) {
      if (isSongChart(chart)) items.push(chart);
    }
  }
  items.sort((a, b) => (a.sort_order ?? 10_000) - (b.sort_order ?? 10_000));
  return items;
});

const filtered = computed(() => {
  const needle = query.value.trim().toLowerCase();
  return props.groups
    .map((group) => ({
      ...group,
      charts: group.charts
        .filter((chart) => isSongChart(chart) && (!needle || chart.name.toLowerCase().includes(needle)))
        .slice()
        .sort((a, b) => (a.sort_order ?? 10_000) - (b.sort_order ?? 10_000)),
    }))
    .filter((group) => group.charts.length);
});

function indexOf(key: string): number {
  return ordered.value.findIndex((chart) => chart.key === key);
}

function onDocClick(event: MouseEvent) {
  if (!root.value?.contains(event.target as Node)) {
    open.value = false;
  }
}

function choose(key: string, playable: boolean) {
  if (!playable) return;
  emit("select", key);
  open.value = false;
  query.value = "";
}

function move(key: string, direction: "up" | "down") {
  emit("move", key, direction);
}

function isCurrent(key: string): boolean {
  return key === props.chartKey;
}

function moveBtnKlass(selected: boolean): string {
  return [
    "grid h-7 w-7 place-items-center rounded-lg transition",
    selected
      ? "bg-white/15 text-white hover:bg-white/25 disabled:bg-transparent disabled:text-white/25 dark:bg-zinc-900/10 dark:text-zinc-600 dark:hover:bg-zinc-900/15 dark:hover:text-zinc-900 dark:disabled:bg-transparent dark:disabled:text-zinc-400"
      : "bg-zinc-100 text-zinc-500 hover:bg-zinc-200 hover:text-zinc-900 disabled:bg-transparent disabled:text-zinc-300 dark:bg-white/10 dark:text-zinc-400 dark:hover:bg-white/15 dark:hover:text-white dark:disabled:bg-transparent dark:disabled:text-zinc-600",
  ].join(" ");
}

onMounted(() => document.addEventListener("click", onDocClick));
onUnmounted(() => document.removeEventListener("click", onDocClick));
</script>

<template>
  <div ref="root" class="relative min-w-0">
    <button
      type="button"
      class="inline-flex min-h-11 max-w-full items-center gap-1 rounded-full px-1 text-left text-lg font-semibold hover:bg-zinc-100 dark:hover:bg-white/10"
      :aria-expanded="open"
      @click="open = !open"
    >
      <span class="truncate">{{ name }}</span>
      <span class="text-sm text-zinc-400" aria-hidden="true">▾</span>
    </button>
    <div
      v-if="open"
      class="absolute left-0 top-full z-30 mt-1 w-[min(100vw-2rem,22rem)] overflow-hidden rounded-2xl bg-white shadow-xl ring-1 ring-zinc-200 dark:bg-zinc-900 dark:ring-white/10"
    >
      <div class="border-b border-zinc-100 p-2 dark:border-white/10">
        <input
          v-model="query"
          type="search"
          placeholder="搜索榜单"
          class="h-11 w-full rounded-full bg-zinc-100 px-3 text-sm outline-none dark:bg-zinc-800"
        />
      </div>
      <div class="max-h-80 overflow-y-auto py-1">
        <div v-for="group in filtered" :key="group.name" class="px-1 py-1">
          <p class="px-3 py-1 text-xs text-zinc-400">{{ group.name }}</p>
          <div
            v-for="chart in group.charts"
            :key="chart.key"
            class="flex items-center gap-1 rounded-xl"
            :class="
              isCurrent(chart.key)
                ? 'bg-zinc-900 text-white dark:bg-white dark:text-zinc-900'
                : 'hover:bg-zinc-100 dark:hover:bg-white/10'
            "
          >
            <button
              type="button"
              class="flex min-h-11 min-w-0 flex-1 items-center justify-between gap-3 px-3 text-left text-sm"
              :class="!chart.playable ? 'cursor-not-allowed text-zinc-400' : ''"
              @click="choose(chart.key, chart.playable)"
            >
              <span class="truncate">{{ chart.name }}</span>
              <span v-if="!chart.playable" class="shrink-0 text-xs">非歌曲</span>
            </button>
            <div class="mr-1 flex shrink-0 items-center gap-0.5" @click.stop>
              <button
                type="button"
                :class="moveBtnKlass(isCurrent(chart.key))"
                :disabled="indexOf(chart.key) <= 0"
                aria-label="上移"
                @click.stop="move(chart.key, 'up')"
              >
                <svg viewBox="0 0 20 20" class="h-3.5 w-3.5" fill="currentColor" aria-hidden="true">
                  <path d="M10 3.6 15.6 9.6h-3.2V16.4H7.6V9.6H4.4Z" />
                </svg>
              </button>
              <button
                type="button"
                :class="moveBtnKlass(isCurrent(chart.key))"
                :disabled="indexOf(chart.key) >= ordered.length - 1"
                aria-label="下移"
                @click.stop="move(chart.key, 'down')"
              >
                <svg viewBox="0 0 20 20" class="h-3.5 w-3.5" fill="currentColor" aria-hidden="true">
                  <path d="M10 16.4 4.4 10.4h3.2V3.6h4.8v6.8h3.2Z" />
                </svg>
              </button>
            </div>
          </div>
        </div>
        <p v-if="!filtered.length" class="px-3 py-6 text-center text-sm text-zinc-500">没有匹配的榜</p>
      </div>
    </div>
  </div>
</template>
