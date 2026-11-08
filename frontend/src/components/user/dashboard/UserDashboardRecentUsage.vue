<template>
  <div class="ru-panel">
    <div class="mb-3 flex items-center justify-between gap-2">
      <h2 class="flex items-center gap-2 text-sm font-semibold text-gray-900 dark:text-white">
        <Icon name="clock" size="xs" :stroke-width="2.5" class="text-primary-500" />
        {{ t('dashboard.recentUsage') }}
      </h2>
      <span class="ru-tag">{{ t('dashboard.last7Days') }}</span>
    </div>

    <!-- 加载骨架 -->
    <div v-if="loading" class="space-y-2">
      <div
        v-for="i in 3"
        :key="i"
        class="flex items-center gap-3 rounded-lg bg-gray-50/70 p-3 dark:bg-dark-800/40"
      >
        <div class="h-8 w-8 flex-shrink-0 animate-pulse rounded-lg bg-gray-200 dark:bg-dark-600/60"></div>
        <div class="flex-1 space-y-1.5">
          <div class="h-3 w-28 animate-pulse rounded bg-gray-200 dark:bg-dark-600/60"></div>
          <div class="h-2.5 w-20 animate-pulse rounded bg-gray-200/70 dark:bg-dark-600/40"></div>
        </div>
        <div class="h-4 w-16 animate-pulse rounded bg-gray-200 dark:bg-dark-600/60"></div>
      </div>
    </div>

    <!-- 空态 -->
    <div v-else-if="data.length === 0" class="flex flex-col items-center justify-center gap-2 py-8 text-center">
      <span class="ru-empty-icon">
        <Icon name="inbox" size="md" :stroke-width="1.6" />
      </span>
      <p class="text-sm font-medium text-gray-600 dark:text-dark-300">
        {{ t('dashboard.noUsageRecords') }}
      </p>
      <p class="max-w-xs text-xs text-gray-400 dark:text-dark-500">
        {{ t('dashboard.startUsingApi') }}
      </p>
      <button type="button" class="ru-link mt-1" @click="router.push('/keys')">
        {{ t('dashboard.createApiKey') }}
        <Icon name="arrowRight" size="xs" />
      </button>
    </div>

    <!-- 列表 -->
    <ul v-else class="divide-y divide-gray-100/80 dark:divide-dark-700/60">
      <li v-for="log in data" :key="log.id" class="ru-row">
        <span class="ru-icon">
          <Icon name="beaker" size="sm" :stroke-width="2" />
        </span>

        <div class="min-w-0 flex-1">
          <p class="truncate text-sm font-medium text-gray-900 dark:text-white" :title="log.model">
            {{ log.model }}
          </p>
          <p class="mt-0.5 flex flex-wrap items-center gap-x-2 text-xs text-gray-500 dark:text-dark-400">
            <span>{{ formatDateTime(log.created_at) }}</span>
            <span class="text-gray-300 dark:text-dark-600">·</span>
            <span class="tabular-nums">
              {{ (log.input_tokens + log.output_tokens).toLocaleString() }} tokens
            </span>
          </p>
        </div>

        <div class="shrink-0 text-right">
          <p class="text-sm font-semibold tabular-nums text-green-600 dark:text-green-400" :title="t('dashboard.actual')">
            ${{ formatCost(log.actual_cost) }}
          </p>
          <p class="text-[11px] tabular-nums text-gray-400 dark:text-dark-500" :title="t('dashboard.standard')">
            ${{ formatCost(log.total_cost) }}
          </p>
        </div>
      </li>
    </ul>

    <router-link v-if="!loading && data.length > 0" to="/usage" class="ru-viewall">
      {{ t('dashboard.viewAllUsage') }}
      <Icon name="arrowRight" size="xs" />
    </router-link>
  </div>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'
import Icon from '@/components/icons/Icon.vue'
import { formatDateTime } from '@/utils/format'
import type { UsageLog } from '@/types'

const router = useRouter()

defineProps<{
  data: UsageLog[]
  loading: boolean
}>()
const { t } = useI18n()
const formatCost = (c: number) => c.toFixed(4)
</script>

<style scoped>
.ru-panel {
  border-radius: 0.875rem;
  padding: 0.875rem;
  background: rgb(255 255 255 / 0.76);
  border: 1px solid rgb(255 255 255 / 0.7);
  box-shadow: 0 1px 2px rgba(15, 23, 42, 0.04);
  backdrop-filter: blur(14px) saturate(1.15);
  -webkit-backdrop-filter: blur(14px) saturate(1.15);
}

:root.dark .ru-panel {
  background: rgb(30 41 59 / 0.55);
  border-color: rgb(148 163 184 / 0.12);
}

.ru-tag {
  border-radius: 9999px;
  padding: 0.0625rem 0.5rem;
  font-size: 0.625rem;
  font-weight: 600;
  color: rgb(100 116 139);
  background: rgb(241 245 249);
}

:root.dark .ru-tag {
  color: rgb(148 163 184);
  background: rgb(15 23 42 / 0.6);
}

.ru-row {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  padding: 0.625rem 0.25rem;
  transition: background-color 150ms ease;
}

.ru-row:hover {
  background: rgb(248 250 252 / 0.8);
}

:root.dark .ru-row:hover {
  background: rgb(15 23 42 / 0.35);
}

.ru-icon {
  display: inline-flex;
  height: 2rem;
  width: 2rem;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  border-radius: 0.5rem;
  color: rgb(13 148 136);
  background: rgb(240 253 250);
}

:root.dark .ru-icon {
  color: rgb(94 234 212);
  background: rgb(19 78 74 / 0.45);
}

.ru-empty-icon {
  display: flex;
  height: 2.5rem;
  width: 2.5rem;
  align-items: center;
  justify-content: center;
  border-radius: 0.75rem;
  color: rgb(148 163 184);
  background: rgb(248 250 252);
  border: 1px solid rgb(226 232 240 / 0.8);
}

:root.dark .ru-empty-icon {
  color: rgb(100 116 139);
  background: rgb(15 23 42 / 0.5);
  border-color: rgb(51 65 85 / 0.7);
}

.ru-viewall {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.375rem;
  margin-top: 0.5rem;
  padding-top: 0.625rem;
  border-top: 1px solid rgb(241 245 249);
  font-size: 0.75rem;
  font-weight: 600;
  color: rgb(13 148 136);
  transition: color 150ms ease;
}

:root.dark .ru-viewall {
  border-top-color: rgb(51 65 85 / 0.6);
  color: rgb(94 234 212);
}

.ru-viewall:hover {
  color: rgb(15 118 110);
}

:root.dark .ru-viewall:hover {
  color: rgb(153 246 228);
}

.ru-link {
  display: inline-flex;
  align-items: center;
  gap: 0.25rem;
  font-size: 0.75rem;
  font-weight: 600;
  color: rgb(13 148 136);
}

:root.dark .ru-link {
  color: rgb(94 234 212);
}
</style>
