<template>
  <!-- 仪表盘首屏加载骨架：保持与真实布局一致的栅格，避免内容跳动 -->
  <div class="space-y-6" aria-busy="true" :aria-label="label">
    <!-- KPI 行 1 -->
    <div class="grid grid-cols-2 gap-4 lg:grid-cols-4">
      <div v-for="i in 4" :key="`r1-${i}`" class="dash-skeleton-card">
        <div class="flex items-start gap-3">
          <div class="h-10 w-10 flex-shrink-0 animate-pulse rounded-xl bg-gray-200 dark:bg-dark-600/60"></div>
          <div class="min-w-0 flex-1 space-y-2">
            <div class="h-3 w-16 animate-pulse rounded bg-gray-200 dark:bg-dark-600/60"></div>
            <div class="h-7 w-24 animate-pulse rounded-md bg-gray-200 dark:bg-dark-600/60"></div>
            <div class="h-3 w-20 animate-pulse rounded bg-gray-200/80 dark:bg-dark-600/40"></div>
          </div>
        </div>
      </div>
    </div>

    <!-- KPI 行 2 -->
    <div class="grid grid-cols-2 gap-4 lg:grid-cols-4">
      <div v-for="i in 4" :key="`r2-${i}`" class="dash-skeleton-card">
        <div class="flex items-start gap-3">
          <div class="h-10 w-10 flex-shrink-0 animate-pulse rounded-xl bg-gray-200 dark:bg-dark-600/60"></div>
          <div class="min-w-0 flex-1 space-y-2">
            <div class="h-3 w-16 animate-pulse rounded bg-gray-200 dark:bg-dark-600/60"></div>
            <div class="h-7 w-20 animate-pulse rounded-md bg-gray-200 dark:bg-dark-600/60"></div>
            <div class="h-3 w-24 animate-pulse rounded bg-gray-200/80 dark:bg-dark-600/40"></div>
          </div>
        </div>
      </div>
    </div>

    <!-- 快捷操作 -->
    <div class="dash-skeleton-card">
      <div class="mb-3 h-4 w-20 animate-pulse rounded bg-gray-200 dark:bg-dark-600/60"></div>
      <div class="grid grid-cols-1 gap-3 md:grid-cols-2">
        <div v-for="i in 2" :key="`qa-${i}`" class="h-16 animate-pulse rounded-lg bg-gray-100 dark:bg-dark-700/40"></div>
      </div>
    </div>

    <!-- 筛选条 -->
    <div class="dash-skeleton-card">
      <div class="flex flex-wrap items-center gap-4">
        <div class="h-9 w-40 animate-pulse rounded-lg bg-gray-200 dark:bg-dark-600/60"></div>
        <div class="h-9 w-20 animate-pulse rounded-lg bg-gray-200 dark:bg-dark-600/60"></div>
        <div class="ml-auto h-9 w-28 animate-pulse rounded-lg bg-gray-200 dark:bg-dark-600/60"></div>
      </div>
    </div>

    <!-- 图表 -->
    <div class="grid grid-cols-1 gap-6 lg:grid-cols-2">
      <div v-for="i in 2" :key="`chart-${i}`" class="dash-skeleton-card">
        <div class="mb-4 h-4 w-32 animate-pulse rounded bg-gray-200 dark:bg-dark-600/60"></div>
        <div class="h-64 animate-pulse rounded-lg bg-gray-100 dark:bg-dark-700/40"></div>
      </div>
    </div>

    <div class="dash-skeleton-card">
      <div class="mb-4 h-4 w-40 animate-pulse rounded bg-gray-200 dark:bg-dark-600/60"></div>
      <div class="h-64 animate-pulse rounded-lg bg-gray-100 dark:bg-dark-700/40"></div>
    </div>
  </div>
</template>

<script setup lang="ts">
interface Props {
  /** 供读屏软件播报的加载提示 */
  label?: string
}

withDefaults(defineProps<Props>(), {
  label: 'Loading dashboard'
})
</script>

<style scoped>
.dash-skeleton-card {
  border-radius: 1rem;
  padding: 1rem;
  background: rgb(255 255 255 / 0.72);
  border: 1px solid rgb(255 255 255 / 0.7);
  box-shadow: 0 1px 2px rgba(15, 23, 42, 0.04);
  backdrop-filter: blur(14px) saturate(1.15);
  -webkit-backdrop-filter: blur(14px) saturate(1.15);
}

:root.dark .dash-skeleton-card {
  background: rgb(30 41 59 / 0.55);
  border-color: rgb(148 163 184 / 0.12);
}
</style>
