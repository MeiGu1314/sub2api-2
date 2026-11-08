<template>
  <AppLayout>
    <div class="dash-page space-y-6">
      <!-- ============ 首次加载：骨架屏 ============ -->
      <DashboardSkeleton v-if="loading && !stats" />

      <!-- ============ 首次加载失败：整页错误态 ============ -->
      <DashboardErrorState
        v-else-if="statsError && !stats"
        variant="page"
        :message="statsError"
        :retrying="loading"
        @retry="loadDashboardStats"
      >
        <template #actions>
          <button type="button" class="dash-secondary-btn" @click="router.push('/admin/ops')">
            {{ t('admin.dashboard.viewAllUsage') }}
          </button>
        </template>
      </DashboardErrorState>

      <!-- ============ 正常内容 ============ -->
      <template v-else-if="stats">
        <!-- Hero 头部 -->
        <section class="dash-hero">
          <span class="dash-hero-orb" aria-hidden="true"></span>
          <span class="dash-hero-grid" aria-hidden="true"></span>

          <div class="relative flex flex-wrap items-start justify-between gap-4">
            <div class="min-w-0">
              <div class="mb-2 flex flex-wrap items-center gap-2">
                <span class="dash-hero-chip">
                  <Icon name="sparkles" size="xs" :stroke-width="2" />
                  {{ t('admin.dashboard.title') }}
                </span>
                <span
                  v-if="stats.stats_stale"
                  class="dash-pill-warn"
                  :title="t('admin.dashboard.staleHint')"
                >
                  {{ t('admin.dashboard.staleBadge') }}
                </span>
              </div>

              <h1 class="dash-hero-title">{{ t('admin.dashboard.title') }}</h1>
              <p class="dash-hero-desc">{{ t('admin.dashboard.description') }}</p>

              <div class="dash-hero-meta">
                <span class="dash-meta-item">
                  <Icon name="clock" size="xs" :stroke-width="2" />
                  <span>{{ t('admin.dashboard.lastUpdatedLabel') }} {{ formattedUpdatedAt }}</span>
                </span>
                <span class="dash-meta-dot" aria-hidden="true"></span>
                <span class="dash-meta-item">
                  <Icon name="bolt" size="xs" :stroke-width="2" />
                  <span class="tabular-nums">{{ formatTokens(stats.rpm) }} RPM</span>
                </span>
                <span class="dash-meta-dot" aria-hidden="true"></span>
                <span class="dash-meta-item">
                  <Icon name="users" size="xs" :stroke-width="2" />
                  <span class="tabular-nums">{{ stats.active_users }}</span>
                </span>
              </div>
            </div>

            <button
              type="button"
              class="dash-refresh-btn"
              :disabled="chartsLoading || loading"
              @click="loadDashboardStats"
            >
              <Icon
                name="refresh"
                size="sm"
                :stroke-width="2"
                :class="(chartsLoading || loading) && 'animate-spin'"
              />
              <span>
                {{ chartsLoading || loading ? t('admin.dashboard.refreshing') : t('common.refresh') }}
              </span>
            </button>
          </div>
        </section>

        <!-- 局部刷新失败横幅 -->
        <DashboardErrorState
          v-if="chartsError"
          variant="inline"
          :message="t('admin.dashboard.errorState.chartsFailed')"
          :retrying="chartsLoading"
          @retry="loadChartData"
        />

        <!-- ============ 核心指标 ============ -->
        <section class="dash-section">
          <header class="dash-section-head">
            <span class="dash-section-icon">
              <Icon name="grid" size="xs" :stroke-width="2.5" />
            </span>
            <h2 class="dash-section-title">{{ t('admin.dashboard.sectionCore') }}</h2>
            <span class="dash-section-line" aria-hidden="true"></span>
          </header>

          <div class="grid grid-cols-2 gap-4 lg:grid-cols-4">
            <DashboardKpiCard
              icon="key"
              tone="blue"
              :label="t('admin.dashboard.apiKeys')"
              :value="stats.total_api_keys"
              :progress="apiKeyActiveRatio"
            >
              <template #meta>
                <span class="text-green-600 dark:text-green-400">
                  {{ t('admin.dashboard.activeRatio', {
                    active: stats.active_api_keys,
                    total: stats.total_api_keys
                  }) }}
                </span>
              </template>
            </DashboardKpiCard>

            <DashboardKpiCard
              icon="server"
              tone="purple"
              :label="t('admin.dashboard.accounts')"
              :value="stats.total_accounts"
              :progress="accountHealthyRatio"
            >
              <template #meta>
                <span class="text-green-600 dark:text-green-400">
                  {{ stats.normal_accounts }} {{ t('admin.dashboard.ok') }}
                </span>
                <span v-if="stats.error_accounts > 0" class="text-red-500">
                  {{ stats.error_accounts }} {{ t('admin.dashboard.err') }}
                </span>
                <span v-if="stats.ratelimit_accounts > 0" class="text-amber-500">
                  {{ stats.ratelimit_accounts }} {{ t('admin.dashboard.performance') }}
                </span>
              </template>
            </DashboardKpiCard>

            <DashboardKpiCard
              icon="chart"
              tone="green"
              :label="t('admin.dashboard.todayRequests')"
              :value="formatNumber(stats.today_requests)"
            >
              <template #meta>
                <span class="tabular-nums">
                  {{ t('admin.dashboard.totalShort') }} {{ formatNumber(stats.total_requests) }}
                </span>
              </template>
            </DashboardKpiCard>

            <DashboardKpiCard
              icon="userPlus"
              tone="emerald"
              :label="t('admin.dashboard.newUsersToday')"
              :value="`+${formatNumber(stats.today_new_users)}`"
            >
              <template #meta>
                <span class="tabular-nums">
                  {{ t('admin.dashboard.totalUsers') }} {{ formatNumber(stats.total_users) }}
                </span>
                <span class="dash-dot" aria-hidden="true"></span>
                <span class="tabular-nums text-emerald-600 dark:text-emerald-400">
                  {{ stats.active_users }} {{ t('admin.dashboard.activeUsers') }}
                </span>
              </template>
            </DashboardKpiCard>
          </div>
        </section>

        <!-- ============ 用量与成本 ============ -->
        <section class="dash-section">
          <header class="dash-section-head">
            <span class="dash-section-icon">
              <Icon name="cube" size="xs" :stroke-width="2.5" />
            </span>
            <h2 class="dash-section-title">{{ t('admin.dashboard.sectionUsage') }}</h2>
            <span class="dash-section-line" aria-hidden="true"></span>
          </header>

          <div class="grid grid-cols-2 gap-4 lg:grid-cols-4">
            <DashboardKpiCard
              icon="cube"
              tone="amber"
              :label="t('admin.dashboard.todayTokens')"
              :value="formatTokens(stats.today_tokens)"
            >
              <template #meta>
                <span class="text-green-600 dark:text-green-400" :title="t('admin.dashboard.actual')">
                  ${{ formatCost(stats.today_actual_cost) }}
                </span>
                <span class="dash-slash">/</span>
                <span class="text-orange-500 dark:text-orange-400" :title="t('admin.dashboard.accountCost')">
                  ${{ formatCost(stats.today_account_cost) }}
                </span>
                <span class="dash-slash">/</span>
                <span class="text-gray-400 dark:text-dark-500" :title="t('admin.dashboard.standard')">
                  ${{ formatCost(stats.today_cost) }}
                </span>
              </template>
            </DashboardKpiCard>

            <DashboardKpiCard
              icon="database"
              tone="indigo"
              :label="t('admin.dashboard.totalTokens')"
              :value="formatTokens(stats.total_tokens)"
            >
              <template #meta>
                <span class="text-green-600 dark:text-green-400" :title="t('admin.dashboard.actual')">
                  ${{ formatCost(stats.total_actual_cost) }}
                </span>
                <span class="dash-slash">/</span>
                <span class="text-orange-500 dark:text-orange-400" :title="t('admin.dashboard.accountCost')">
                  ${{ formatCost(stats.total_account_cost) }}
                </span>
                <span class="dash-slash">/</span>
                <span class="text-gray-400 dark:text-dark-500" :title="t('admin.dashboard.standard')">
                  ${{ formatCost(stats.total_cost) }}
                </span>
              </template>
            </DashboardKpiCard>

            <DashboardKpiCard
              icon="bolt"
              tone="violet"
              :label="t('admin.dashboard.performance')"
              :value="formatTokens(stats.rpm)"
              :accent="formatTokens(stats.tpm)"
              accent-label="TPM"
            >
              <template #meta>
                <span class="text-xs text-gray-400 dark:text-dark-500">RPM</span>
              </template>
            </DashboardKpiCard>

            <DashboardKpiCard
              icon="clock"
              tone="rose"
              :label="t('admin.dashboard.avgResponse')"
              :value="formatDuration(stats.average_duration_ms)"
            >
              <template #meta>
                <span class="tabular-nums">
                  {{ stats.active_users }} {{ t('admin.dashboard.activeUsers') }}
                </span>
              </template>
            </DashboardKpiCard>
          </div>
        </section>

        <!-- ============ 快捷操作 ============ -->
        <section class="dash-section">
          <header class="dash-section-head">
            <span class="dash-section-icon">
              <Icon name="bolt" size="xs" :stroke-width="2.5" />
            </span>
            <h2 class="dash-section-title">{{ t('admin.dashboard.quickActions') }}</h2>
            <span class="dash-section-line" aria-hidden="true"></span>
          </header>

          <div class="grid grid-cols-1 gap-3 md:grid-cols-2">
            <button
              v-if="canUseBatchImage"
              type="button"
              class="dash-action dash-action-sky group"
              @click="router.push('/batch-image')"
            >
              <span class="dash-action-icon">
                <Icon name="sparkles" size="md" :stroke-width="2" />
              </span>
              <span class="min-w-0 flex-1">
                <span class="block text-sm font-semibold text-gray-900 dark:text-white">
                  {{ t('admin.dashboard.batchImage') }}
                </span>
                <span class="mt-0.5 block text-xs text-gray-500 dark:text-dark-400">
                  {{ t('admin.dashboard.batchImageDesc') }}
                </span>
              </span>
              <Icon
                name="chevronRight"
                size="sm"
                class="text-gray-400 transition-transform duration-200 group-hover:translate-x-0.5 group-hover:text-sky-500"
              />
            </button>

            <button
              type="button"
              class="dash-action dash-action-emerald group"
              @click="router.push('/admin/groups')"
            >
              <span class="dash-action-icon">
                <Icon name="grid" size="md" :stroke-width="2" />
              </span>
              <span class="min-w-0 flex-1">
                <span class="block text-sm font-semibold text-gray-900 dark:text-white">
                  {{ t('admin.dashboard.groupPricing') }}
                </span>
                <span class="mt-0.5 block text-xs text-gray-500 dark:text-dark-400">
                  {{ t('admin.dashboard.groupPricingDesc') }}
                </span>
              </span>
              <Icon
                name="chevronRight"
                size="sm"
                class="text-gray-400 transition-transform duration-200 group-hover:translate-x-0.5 group-hover:text-emerald-500"
              />
            </button>
          </div>
        </section>

        <!-- ============ 趋势与榜单 ============ -->
        <section class="dash-section">
          <header class="dash-section-head">
            <span class="dash-section-icon">
              <Icon name="trendingUp" size="xs" :stroke-width="2.5" />
            </span>
            <h2 class="dash-section-title">{{ t('admin.dashboard.sectionTrends') }}</h2>
            <span class="dash-section-line" aria-hidden="true"></span>
          </header>

          <!-- 筛选工具条 -->
          <div class="dash-toolbar">
            <div class="flex flex-wrap items-center gap-3">
              <div class="flex items-center gap-2">
                <span class="dash-toolbar-label">{{ t('admin.dashboard.timeRange') }}</span>
                <DateRangePicker
                  v-model:start-date="startDate"
                  v-model:end-date="endDate"
                  @change="onDateRangeChange"
                />
              </div>

              <span class="dash-vr" aria-hidden="true"></span>

              <div class="flex items-center gap-2">
                <span class="dash-toolbar-label">{{ t('admin.dashboard.granularity') }}</span>
                <div class="w-28">
                  <Select
                    v-model="granularity"
                    :options="granularityOptions"
                    @change="loadChartData"
                  />
                </div>
              </div>
            </div>
          </div>

          <!-- 图表栅格 -->
          <div class="dash-glass grid grid-cols-1 gap-4 lg:grid-cols-2">
            <DashboardChartPlaceholder
              v-if="chartsLoading && modelStats.length === 0"
              height-class="h-80"
              loading
            />
            <DashboardChartPlaceholder
              v-else-if="modelStats.length === 0"
              height-class="h-80"
              icon="chartBar"
              :title="t('admin.dashboard.emptyState.chartNoData')"
              :description="t('admin.dashboard.emptyState.chartNoDataHint')"
              :action-text="t('admin.dashboard.emptyState.resetRange')"
              @action="resetDateRange"
            />
            <ModelDistributionChart
              v-else
              :model-stats="modelStats"
              :enable-ranking-view="true"
              :ranking-items="rankingItems"
              :ranking-total-actual-cost="rankingTotalActualCost"
              :ranking-total-requests="rankingTotalRequests"
              :ranking-total-tokens="rankingTotalTokens"
              :loading="chartsLoading"
              :ranking-loading="rankingLoading"
              :ranking-error="rankingError"
              :start-date="startDate"
              :end-date="endDate"
              @ranking-click="goToUserUsage"
            />

            <TokenUsageTrend
              :trend-data="trendData"
              :loading="chartsLoading && trendData.length === 0"
            />
          </div>

          <!-- 用户使用趋势 -->
          <div class="dash-panel p-4">
            <div class="mb-4 flex flex-wrap items-center justify-between gap-3">
              <h3 class="flex items-center gap-2 text-sm font-semibold text-gray-900 dark:text-white">
                <Icon name="users" size="xs" :stroke-width="2.5" class="text-primary-500" />
                {{ t('admin.dashboard.userUsageTrend') }}
              </h3>

              <div
                class="dash-segmented"
                role="group"
                :aria-label="t('admin.dashboard.userUsageTrend')"
              >
                <button
                  v-for="metric in (['tokens', 'actual_cost'] as const)"
                  :key="metric"
                  type="button"
                  class="dash-segmented-item"
                  :class="userTrendMetric === metric && 'dash-segmented-item-active'"
                  :aria-pressed="userTrendMetric === metric"
                  @click="setUserTrendMetric(metric)"
                >
                  {{
                    t(metric === 'tokens' ? 'admin.dashboard.tokens' : 'admin.dashboard.actualSpending')
                  }}
                </button>
              </div>
            </div>

            <DashboardChartPlaceholder
              v-if="userTrendLoading && userTrend.length === 0"
              loading
            />
            <DashboardChartPlaceholder
              v-else-if="!userTrendChartData"
              icon="users"
              :title="t('admin.dashboard.emptyState.usersNoData')"
              :description="t('admin.dashboard.emptyState.usersNoDataHint')"
            />
            <div v-else class="h-64">
              <Line :data="userTrendChartData" :options="lineOptions" />
            </div>
          </div>
        </section>
      </template>
    </div>
  </AppLayout>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRouter } from 'vue-router'
import { useAppStore } from '@/stores/app'

const { t } = useI18n()
import { adminAPI } from '@/api/admin'
import type {
  DashboardStats,
  TrendDataPoint,
  ModelStat,
  UserUsageTrendPoint,
  UserSpendingRankingItem
} from '@/types'
import AppLayout from '@/components/layout/AppLayout.vue'
import Icon from '@/components/icons/Icon.vue'
import DateRangePicker from '@/components/common/DateRangePicker.vue'
import Select from '@/components/common/Select.vue'
import ModelDistributionChart from '@/components/charts/ModelDistributionChart.vue'
import TokenUsageTrend from '@/components/charts/TokenUsageTrend.vue'
import DashboardKpiCard from '@/components/admin/dashboard/DashboardKpiCard.vue'
import DashboardSkeleton from '@/components/admin/dashboard/DashboardSkeleton.vue'
import DashboardErrorState from '@/components/admin/dashboard/DashboardErrorState.vue'
import DashboardChartPlaceholder from '@/components/admin/dashboard/DashboardChartPlaceholder.vue'
import { useBatchImageAccess } from '@/composables/useBatchImageAccess'

import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Tooltip,
  Legend,
  Filler
} from 'chart.js'
import { Line } from 'vue-chartjs'

// Register Chart.js components
ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Tooltip,
  Legend,
  Filler
)

const appStore = useAppStore()
const router = useRouter()
const { canUseBatchImage, refreshBatchImageAccess } = useBatchImageAccess()
const stats = ref<DashboardStats | null>(null)
const loading = ref(false)
const chartsLoading = ref(false)
const userTrendLoading = ref(false)
const rankingLoading = ref(false)
const rankingError = ref(false)

// 状态与错误
const statsError = ref<string | null>(null)
const chartsError = ref(false)
const updatedAt = ref<Date | null>(null)
// 每 30s 刷新一次“x 分钟前”的相对时间展示
const nowTick = ref(Date.now())
let timeTicker: number | undefined

// Chart data
const trendData = ref<TrendDataPoint[]>([])
const modelStats = ref<ModelStat[]>([])
const userTrend = ref<UserUsageTrendPoint[]>([])
const userTrendMetric = ref<'tokens' | 'actual_cost'>('tokens')
const rankingItems = ref<UserSpendingRankingItem[]>([])
const rankingTotalActualCost = ref(0)
const rankingTotalRequests = ref(0)
const rankingTotalTokens = ref(0)
let chartLoadSeq = 0
let usersTrendLoadSeq = 0
let rankingLoadSeq = 0
const rankingLimit = 12

// Helper function to format date in local timezone
const formatLocalDate = (date: Date): string => {
  return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`
}

const getLast24HoursRangeDates = (): { start: string; end: string } => {
  const end = new Date()
  const start = new Date(end.getTime() - 24 * 60 * 60 * 1000)
  return {
    start: formatLocalDate(start),
    end: formatLocalDate(end)
  }
}

// Date range
const granularity = ref<'day' | 'hour'>('hour')
const defaultRange = getLast24HoursRangeDates()
const startDate = ref(defaultRange.start)
const endDate = ref(defaultRange.end)

const resetDateRange = () => {
  const range = getLast24HoursRangeDates()
  startDate.value = range.start
  endDate.value = range.end
  granularity.value = 'hour'
  loadChartData()
}

// Granularity options for Select component
const granularityOptions = computed(() => [
  { value: 'day', label: t('admin.dashboard.day') },
  { value: 'hour', label: t('admin.dashboard.hour') }
])

// Dark mode detection
const isDarkMode = computed(() => {
  return document.documentElement.classList.contains('dark')
})

// Chart colors
const chartColors = computed(() => ({
  text: isDarkMode.value ? '#e5e7eb' : '#374151',
  grid: isDarkMode.value ? '#374151' : '#e5e7eb'
}))

// ============ 派生指标 ============

/** 活跃密钥占比（0-100），用于底部进度条 */
const apiKeyActiveRatio = computed(() => {
  const total = Number(stats.value?.total_api_keys) || 0
  const active = Number(stats.value?.active_api_keys) || 0
  if (total <= 0) return 0
  return (active / total) * 100
})

/** 账号健康占比（0-100） */
const accountHealthyRatio = computed(() => {
  const total = Number(stats.value?.total_accounts) || 0
  const normal = Number(stats.value?.normal_accounts) || 0
  if (total <= 0) return 0
  return (normal / total) * 100
})

/** 统计更新时间的友好展示 */
const formattedUpdatedAt = computed(() => {
  const time = updatedAt.value
  if (!time) return '—'
  const diffMs = nowTick.value - time.getTime()
  if (!Number.isFinite(diffMs) || diffMs < 0) return t('admin.dashboard.justNow')
  const minutes = Math.floor(diffMs / 60_000)
  if (minutes < 1) return t('admin.dashboard.justNow')
  if (minutes < 60) return t('admin.dashboard.minutesAgo', { count: minutes })
  const hours = Math.floor(minutes / 60)
  if (hours < 24) return t('admin.dashboard.hoursAgo', { count: hours })
  return time.toLocaleString()
})

// Line chart options (for user trend chart)
const lineOptions = computed(() => ({
  responsive: true,
  maintainAspectRatio: false,
  interaction: {
    intersect: false,
    mode: 'index' as const
  },
  plugins: {
    legend: {
      position: 'top' as const,
      labels: {
        color: chartColors.value.text,
        usePointStyle: true,
        pointStyle: 'circle',
        padding: 15,
        font: {
          size: 11
        }
      }
    },
    tooltip: {
      itemSort: (a: any, b: any) => {
        const aValue = typeof a?.raw === 'number' ? a.raw : Number(a?.parsed?.y ?? 0)
        const bValue = typeof b?.raw === 'number' ? b.raw : Number(b?.parsed?.y ?? 0)
        return bValue - aValue
      },
      callbacks: {
        label: (context: any) => {
          return `${context.dataset.label}: ${formatUserTrendValue(Number(context.raw))}`
        }
      }
    }
  },
  scales: {
    x: {
      grid: {
        color: chartColors.value.grid
      },
      ticks: {
        color: chartColors.value.text,
        font: {
          size: 10
        }
      }
    },
    y: {
      grid: {
        color: chartColors.value.grid
      },
      ticks: {
        color: chartColors.value.text,
        font: {
          size: 10
        },
        callback: (value: string | number) => formatUserTrendValue(Number(value))
      }
    }
  }
}))

// User trend chart data
const userTrendChartData = computed(() => {
  if (!userTrend.value?.length) return null

  const getDisplayName = (point: UserUsageTrendPoint): string => {
    const username = point.username?.trim()
    if (username) {
      return username
    }

    const email = point.email?.trim()
    if (email) {
      return email
    }

    return t('admin.redeem.userPrefix', { id: point.user_id })
  }

  // Group by user_id to avoid merging different users with the same display name
  const userGroups = new Map<number, { name: string; data: Map<string, number> }>()
  const allDates = new Set<string>()

  userTrend.value.forEach((point) => {
    allDates.add(point.date)
    const key = point.user_id
    if (!userGroups.has(key)) {
      userGroups.set(key, { name: getDisplayName(point), data: new Map() })
    }
    userGroups.get(key)!.data.set(point.date, userTrendMetric.value === 'tokens' ? point.tokens : point.actual_cost)
  })

  const sortedDates = Array.from(allDates).sort()
  const colors = [
    '#3b82f6',
    '#10b981',
    '#f59e0b',
    '#ef4444',
    '#8b5cf6',
    '#ec4899',
    '#14b8a6',
    '#f97316',
    '#6366f1',
    '#84cc16',
    '#06b6d4',
    '#a855f7'
  ]

  const datasets = Array.from(userGroups.values()).map((group, idx) => ({
    label: group.name,
    data: sortedDates.map((date) => group.data.get(date) || 0),
    borderColor: colors[idx % colors.length],
    backgroundColor: `${colors[idx % colors.length]}20`,
    fill: false,
    tension: 0.3
  }))

  return {
    labels: sortedDates,
    datasets
  }
})

// Format helpers
const formatUserTrendValue = (value: number): string =>
  userTrendMetric.value === 'tokens' ? formatTokens(value) : `$${formatCost(value)}`

const formatTokens = (value: number | undefined): string => {
  if (value === undefined || value === null) return '0'
  if (value >= 1_000_000_000) {
    return `${(value / 1_000_000_000).toFixed(2)}B`
  } else if (value >= 1_000_000) {
    return `${(value / 1_000_000).toFixed(2)}M`
  } else if (value >= 1_000) {
    return `${(value / 1_000).toFixed(2)}K`
  }
  return value.toLocaleString()
}

const toFiniteNumber = (value: unknown): number => {
  const numberValue = Number(value)
  return Number.isFinite(numberValue) ? numberValue : 0
}

const formatNumber = (value: number | null | undefined): string => {
  return toFiniteNumber(value).toLocaleString()
}

const formatCost = (value: number | null | undefined): string => {
  const safeValue = toFiniteNumber(value)
  if (safeValue >= 1000) {
    return (safeValue / 1000).toFixed(2) + 'K'
  } else if (safeValue >= 1) {
    return safeValue.toFixed(2)
  } else if (safeValue >= 0.01) {
    return safeValue.toFixed(3)
  }
  return safeValue.toFixed(4)
}

const formatDuration = (ms: number): string => {
  if (ms >= 1000) {
    return `${(ms / 1000).toFixed(2)}s`
  }
  return `${Math.round(ms)}ms`
}

const goToUserUsage = (item: UserSpendingRankingItem) => {
  void router.push({
    path: '/admin/usage',
    query: {
      user_id: String(item.user_id),
      start_date: startDate.value,
      end_date: endDate.value
    }
  })
}

// Date range change handler
const onDateRangeChange = (range: {
  startDate: string
  endDate: string
  preset: string | null
}) => {
  // Auto-select granularity based on date range
  const start = new Date(range.startDate)
  const end = new Date(range.endDate)
  const daysDiff = Math.ceil((end.getTime() - start.getTime()) / (1000 * 60 * 60 * 24))

  // If range is 1 day, use hourly granularity
  if (daysDiff <= 1) {
    granularity.value = 'hour'
  } else {
    granularity.value = 'day'
  }

  loadChartData()
}

// Load data
const loadDashboardSnapshot = async (includeStats: boolean) => {
  const currentSeq = ++chartLoadSeq
  if (includeStats && !stats.value) {
    loading.value = true
  }
  chartsLoading.value = true
  try {
    const response = await adminAPI.dashboard.getSnapshotV2({
      start_date: startDate.value,
      end_date: endDate.value,
      granularity: granularity.value,
      include_stats: includeStats,
      include_trend: true,
      include_model_stats: true,
      include_group_stats: false,
      include_users_trend: false
    })
    if (currentSeq !== chartLoadSeq) return
    if (includeStats && response.stats) {
      stats.value = response.stats
    }
    trendData.value = response.trend || []
    modelStats.value = response.models || []
    // 数据成功返回后清除错误标记
    statsError.value = null
    chartsError.value = false
    updatedAt.value = new Date()
  } catch (error) {
    if (currentSeq !== chartLoadSeq) return
    const message = error instanceof Error ? error.message : ''
    // 已有数据时降级为局部错误；否则为整页错误
    if (includeStats && !stats.value) {
      statsError.value = message || t('admin.dashboard.failedToLoad')
    } else {
      chartsError.value = true
    }
    appStore.showError(t('admin.dashboard.failedToLoad'))
    console.error('Error loading dashboard snapshot:', error)
  } finally {
    if (currentSeq === chartLoadSeq) {
      loading.value = false
      chartsLoading.value = false
    }
  }
}

const setUserTrendMetric = (metric: 'tokens' | 'actual_cost') => {
  if (userTrendMetric.value === metric) return
  userTrendMetric.value = metric
  userTrend.value = []
  loadUsersTrend()
}

const loadUsersTrend = async () => {
  const currentSeq = ++usersTrendLoadSeq
  userTrendLoading.value = true
  try {
    const response = await adminAPI.dashboard.getUserUsageTrend({
      start_date: startDate.value,
      end_date: endDate.value,
      granularity: granularity.value,
      limit: 12,
      metric: userTrendMetric.value
    })
    if (currentSeq !== usersTrendLoadSeq) return
    userTrend.value = response.trend || []
  } catch (error) {
    if (currentSeq !== usersTrendLoadSeq) return
    console.error('Error loading users trend:', error)
    userTrend.value = []
    chartsError.value = true
  } finally {
    if (currentSeq === usersTrendLoadSeq) {
      userTrendLoading.value = false
    }
  }
}

const loadUserSpendingRanking = async () => {
  const currentSeq = ++rankingLoadSeq
  rankingLoading.value = true
  rankingError.value = false
  try {
    const response = await adminAPI.dashboard.getUserSpendingRanking({
      start_date: startDate.value,
      end_date: endDate.value,
      limit: rankingLimit
    })
    if (currentSeq !== rankingLoadSeq) return
    rankingItems.value = response.ranking || []
    rankingTotalActualCost.value = response.total_actual_cost || 0
    rankingTotalRequests.value = response.total_requests || 0
    rankingTotalTokens.value = response.total_tokens || 0
  } catch (error) {
    if (currentSeq !== rankingLoadSeq) return
    console.error('Error loading user spending ranking:', error)
    rankingItems.value = []
    rankingTotalActualCost.value = 0
    rankingTotalRequests.value = 0
    rankingTotalTokens.value = 0
    rankingError.value = true
  } finally {
    if (currentSeq === rankingLoadSeq) {
      rankingLoading.value = false
    }
  }
}

const loadDashboardStats = async () => {
  await Promise.all([
    loadDashboardSnapshot(true),
    loadUsersTrend(),
    loadUserSpendingRanking()
  ])
}

const loadChartData = async () => {
  await Promise.all([
    loadDashboardSnapshot(false),
    loadUsersTrend(),
    loadUserSpendingRanking()
  ])
}

onMounted(() => {
  void refreshBatchImageAccess()
  loadDashboardStats()
  // 相对时间展示的计时器；页面卸载时清除，避免泄漏
  timeTicker = window.setInterval(() => {
    nowTick.value = Date.now()
  }, 30_000)
})

onBeforeUnmount(() => {
  if (timeTicker !== undefined) {
    window.clearInterval(timeTicker)
    timeTicker = undefined
  }
})
</script>

<style scoped>
/* ============================================================
   仪表盘：Hero 头部
   ============================================================ */
.dash-hero {
  position: relative;
  overflow: hidden;
  border-radius: 1.25rem;
  padding: 1.375rem 1.5rem;
  background:
    linear-gradient(135deg, rgba(255, 255, 255, 0.92) 0%, rgba(240, 253, 250, 0.86) 100%);
  border: 1px solid rgb(255 255 255 / 0.7);
  box-shadow: 0 1px 2px rgba(15, 23, 42, 0.04), 0 18px 44px -24px rgba(13, 148, 136, 0.35);
  backdrop-filter: blur(16px) saturate(1.2);
  -webkit-backdrop-filter: blur(16px) saturate(1.2);
}

:root.dark .dash-hero {
  background: linear-gradient(135deg, rgba(30, 41, 59, 0.82) 0%, rgba(19, 78, 74, 0.42) 100%);
  border-color: rgb(148 163 184 / 0.14);
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.35), 0 18px 44px -24px rgba(0, 0, 0, 0.7);
}

.dash-hero-orb {
  position: absolute;
  top: -6rem;
  right: -4rem;
  height: 16rem;
  width: 16rem;
  border-radius: 9999px;
  background: radial-gradient(circle, rgba(20, 184, 166, 0.28) 0%, transparent 70%);
  pointer-events: none;
}

.dash-hero-grid {
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(to right, rgba(13, 148, 136, 0.07) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(13, 148, 136, 0.07) 1px, transparent 1px);
  background-size: 32px 32px;
  -webkit-mask-image: radial-gradient(ellipse 70% 100% at 100% 0%, #000 20%, transparent 75%);
  mask-image: radial-gradient(ellipse 70% 100% at 100% 0%, #000 20%, transparent 75%);
  pointer-events: none;
}

.dash-hero-chip {
  display: inline-flex;
  align-items: center;
  gap: 0.3125rem;
  border-radius: 9999px;
  padding: 0.1875rem 0.5625rem;
  font-size: 0.6875rem;
  font-weight: 600;
  letter-spacing: 0.01em;
  color: rgb(15 118 110);
  background: rgb(255 255 255 / 0.85);
  border: 1px solid rgb(20 184 166 / 0.22);
}

:root.dark .dash-hero-chip {
  color: rgb(94 234 212);
  background: rgb(19 78 74 / 0.55);
  border-color: rgb(20 184 166 / 0.28);
}

.dash-hero-title {
  font-size: 1.5rem;
  line-height: 2rem;
  font-weight: 700;
  letter-spacing: -0.02em;
  color: rgb(15 23 42);
}

:root.dark .dash-hero-title {
  color: rgb(255 255 255);
}

.dash-hero-desc {
  margin-top: 0.25rem;
  font-size: 0.8125rem;
  color: rgb(100 116 139);
}

:root.dark .dash-hero-desc {
  color: rgb(148 163 184);
}

.dash-hero-meta {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 0.5rem;
  margin-top: 0.75rem;
}

.dash-meta-item {
  display: inline-flex;
  align-items: center;
  gap: 0.3125rem;
  border-radius: 0.5rem;
  padding: 0.1875rem 0.5rem;
  font-size: 0.6875rem;
  font-weight: 500;
  color: rgb(71 85 105);
  background: rgb(255 255 255 / 0.7);
  border: 1px solid rgb(226 232 240 / 0.9);
}

:root.dark .dash-meta-item {
  color: rgb(203 213 225);
  background: rgb(15 23 42 / 0.45);
  border-color: rgb(51 65 85 / 0.7);
}

.dash-meta-dot {
  height: 0.1875rem;
  width: 0.1875rem;
  border-radius: 9999px;
  background: rgb(148 163 184);
  opacity: 0.7;
}

.dash-pill-warn {
  display: inline-flex;
  align-items: center;
  border-radius: 9999px;
  padding: 0.125rem 0.5rem;
  font-size: 0.625rem;
  font-weight: 600;
  color: rgb(180 83 9);
  background: rgb(254 243 199);
  border: 1px solid rgb(252 211 77 / 0.7);
}

:root.dark .dash-pill-warn {
  color: rgb(252 211 77);
  background: rgb(120 53 15 / 0.4);
  border-color: rgb(180 83 9 / 0.45);
}

/* ============================================================
   按钮
   ============================================================ */
.dash-refresh-btn {
  display: inline-flex;
  flex-shrink: 0;
  align-items: center;
  gap: 0.375rem;
  border-radius: 0.6875rem;
  padding: 0.5rem 0.875rem;
  font-size: 0.75rem;
  font-weight: 600;
  color: rgb(255 255 255);
  background: linear-gradient(135deg, #14b8a6 0%, #0d9488 100%);
  box-shadow: 0 4px 14px -4px rgba(13, 148, 136, 0.55);
  transition: transform 160ms ease, box-shadow 160ms ease, opacity 160ms ease;
}

.dash-refresh-btn:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 8px 22px -6px rgba(13, 148, 136, 0.65);
}

.dash-refresh-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  box-shadow: none;
}

.dash-secondary-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  border-radius: 0.625rem;
  padding: 0.4375rem 0.8125rem;
  font-size: 0.75rem;
  font-weight: 500;
  color: rgb(51 65 85);
  background: rgb(255 255 255 / 0.9);
  border: 1px solid rgb(226 232 240);
  transition: background-color 150ms ease;
}

.dash-secondary-btn:hover {
  background: rgb(248 250 252);
}

:root.dark .dash-secondary-btn {
  color: rgb(203 213 225);
  background: rgb(30 41 59 / 0.8);
  border-color: rgb(51 65 85);
}

/* ============================================================
   分区标题
   ============================================================ */
.dash-section {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.dash-section-head {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.dash-section-icon {
  display: inline-flex;
  height: 1.375rem;
  width: 1.375rem;
  align-items: center;
  justify-content: center;
  border-radius: 0.4375rem;
  color: rgb(13 148 136);
  background: rgb(240 253 250);
  border: 1px solid rgb(20 184 166 / 0.2);
}

:root.dark .dash-section-icon {
  color: rgb(94 234 212);
  background: rgb(19 78 74 / 0.45);
  border-color: rgb(20 184 166 / 0.25);
}

.dash-section-title {
  font-size: 0.8125rem;
  font-weight: 600;
  letter-spacing: 0.01em;
  color: rgb(51 65 85);
}

:root.dark .dash-section-title {
  color: rgb(203 213 225);
}

.dash-section-line {
  flex: 1;
  height: 1px;
  background: linear-gradient(to right, rgb(203 213 225 / 0.8), transparent);
}

:root.dark .dash-section-line {
  background: linear-gradient(to right, rgb(51 65 85 / 0.9), transparent);
}

/* ============================================================
   玻璃面板 / 工具条
   ============================================================ */
.dash-panel,
.dash-toolbar {
  border-radius: 1rem;
  background: rgb(255 255 255 / 0.72);
  border: 1px solid rgb(255 255 255 / 0.7);
  box-shadow: 0 1px 2px rgba(15, 23, 42, 0.04), 0 10px 28px -18px rgba(15, 23, 42, 0.28);
  backdrop-filter: blur(14px) saturate(1.15);
  -webkit-backdrop-filter: blur(14px) saturate(1.15);
}

:root.dark .dash-panel,
:root.dark .dash-toolbar {
  background: rgb(30 41 59 / 0.55);
  border-color: rgb(148 163 184 / 0.12);
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.3), 0 10px 28px -18px rgba(0, 0, 0, 0.6);
}

.dash-toolbar {
  padding: 0.75rem 1rem;
}

.dash-toolbar-label {
  font-size: 0.75rem;
  font-weight: 500;
  color: rgb(71 85 105);
}

:root.dark .dash-toolbar-label {
  color: rgb(203 213 225);
}

.dash-vr {
  width: 1px;
  height: 1.125rem;
  background: rgb(226 232 240);
}

:root.dark .dash-vr {
  background: rgb(51 65 85);
}

/* 图表子组件内的 .card 统一改为玻璃质感 */
.dash-glass :deep(.card) {
  background: rgb(255 255 255 / 0.72);
  border-color: rgb(255 255 255 / 0.7);
  box-shadow: 0 1px 2px rgba(15, 23, 42, 0.04), 0 10px 28px -18px rgba(15, 23, 42, 0.28);
  backdrop-filter: blur(14px) saturate(1.15);
  -webkit-backdrop-filter: blur(14px) saturate(1.15);
}

:root.dark .dash-glass :deep(.card) {
  background: rgb(30 41 59 / 0.55);
  border-color: rgb(148 163 184 / 0.12);
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.3), 0 10px 28px -18px rgba(0, 0, 0, 0.6);
}

/* ============================================================
   快捷操作
   ============================================================ */
.dash-action {
  position: relative;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  overflow: hidden;
  border-radius: 1rem;
  padding: 0.875rem 1rem;
  text-align: left;
  background: rgb(255 255 255 / 0.72);
  border: 1px solid rgb(255 255 255 / 0.7);
  box-shadow: 0 1px 2px rgba(15, 23, 42, 0.04);
  backdrop-filter: blur(14px) saturate(1.15);
  -webkit-backdrop-filter: blur(14px) saturate(1.15);
  transition: transform 200ms ease, box-shadow 200ms ease, border-color 200ms ease;
}

.dash-action:hover {
  transform: translateY(-2px);
}

:root.dark .dash-action {
  background: rgb(30 41 59 / 0.55);
  border-color: rgb(148 163 184 / 0.12);
}

.dash-action-sky:hover {
  border-color: rgb(56 189 248 / 0.35);
  box-shadow: 0 10px 28px -14px rgba(14, 165, 233, 0.5);
}

.dash-action-emerald:hover {
  border-color: rgb(52 211 153 / 0.35);
  box-shadow: 0 10px 28px -14px rgba(16, 185, 129, 0.5);
}

.dash-action-icon {
  display: flex;
  height: 2.5rem;
  width: 2.5rem;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  border-radius: 0.75rem;
}

.dash-action-sky .dash-action-icon {
  color: rgb(2 132 199);
  background: linear-gradient(135deg, rgb(224 242 254) 0%, rgb(186 230 253) 100%);
}

:root.dark .dash-action-sky .dash-action-icon {
  color: rgb(125 211 252);
  background: linear-gradient(135deg, rgb(12 74 110 / 0.7) 0%, rgb(7 89 133 / 0.5) 100%);
}

.dash-action-emerald .dash-action-icon {
  color: rgb(5 150 105);
  background: linear-gradient(135deg, rgb(209 250 229) 0%, rgb(167 243 208) 100%);
}

:root.dark .dash-action-emerald .dash-action-icon {
  color: rgb(110 231 183);
  background: linear-gradient(135deg, rgb(6 78 59 / 0.7) 0%, rgb(6 95 70 / 0.5) 100%);
}

/* ============================================================
   分段控件
   ============================================================ */
.dash-segmented {
  display: inline-flex;
  gap: 0.125rem;
  border-radius: 0.5625rem;
  padding: 0.1875rem;
  background: rgb(241 245 249);
  border: 1px solid rgb(226 232 240);
}

:root.dark .dash-segmented {
  background: rgb(15 23 42 / 0.6);
  border-color: rgb(51 65 85 / 0.7);
}

.dash-segmented-item {
  border-radius: 0.4375rem;
  padding: 0.1875rem 0.5625rem;
  font-size: 0.6875rem;
  font-weight: 500;
  color: rgb(71 85 105);
  transition: color 150ms ease, background-color 150ms ease;
}

:root.dark .dash-segmented-item {
  color: rgb(203 213 225);
}

.dash-segmented-item:hover {
  color: rgb(15 118 110);
}

.dash-segmented-item-active,
.dash-segmented-item-active:hover {
  color: rgb(255 255 255);
  background: linear-gradient(135deg, #14b8a6 0%, #0d9488 100%);
  box-shadow: 0 1px 3px rgba(13, 148, 136, 0.35);
}

/* ============================================================
   小工具
   ============================================================ */
.dash-dot {
  display: inline-block;
  height: 0.1875rem;
  width: 0.1875rem;
  border-radius: 9999px;
  background: currentColor;
  opacity: 0.4;
}

.dash-slash {
  color: rgb(203 213 225);
}

:root.dark .dash-slash {
  color: rgb(71 85 105);
}
</style>
