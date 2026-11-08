<template>
  <AppLayout>
    <div class="ud-page">
      <!-- 骨架 -->
      <DashboardSkeleton v-if="loading && !stats" />

      <!-- 错误 -->
      <DashboardErrorState
        v-else-if="loadError && !stats"
        variant="page"
        :message="loadError"
        :retrying="loading"
        @retry="refreshAll"
      />

      <!-- 内容 -->
      <template v-else-if="stats">
        <!-- 问候行 -->
        <div class="ud-greet">
          <div class="min-w-0">
            <h1 class="ud-greet-title">
              {{ greeting }}<template v-if="displayName">，{{ displayName }}</template>
            </h1>
            <p class="ud-greet-sub">{{ t('dashboard.welcomeMessage') }}</p>
          </div>
        </div>

        <DashboardErrorState
          v-if="loadError && stats"
          variant="inline"
          :message="loadError"
          :retrying="loading"
          @retry="refreshAll"
        />

        <!-- 主视觉：余额 + 今日焦点 -->
        <DashboardHero
          :balance="user?.balance || 0"
          :is-simple="authStore.isSimpleMode"
          :concurrency="user?.concurrency || 0"
          :rpm="stats.rpm || 0"
          :active-keys="stats.active_api_keys || 0"
          :today-cost="stats.today_actual_cost || 0"
          :today-tokens="stats.today_tokens || 0"
          :today-requests="stats.today_requests || 0"
          :avg-response-ms="stats.average_duration_ms || 0"
          :loading="loading"
          @refresh="refreshAll"
          @navigate="router.push($event)"
        />

        <!-- 新用户引导 -->
        <DashboardGetStarted v-if="!hasAnyUsage" @navigate="router.push($event)" />

        <!-- 统计带（含 Token 构成与平台） -->
        <UserDashboardStats
          :stats="stats"
          :balance="user?.balance || 0"
          :is-simple="authStore.isSimpleMode"
          :platform-quotas="platformQuotas"
          @navigate="router.push($event)"
        />

        <!-- 有数据：图表 + 侧栏 -->
        <div v-if="hasAnyUsage" class="ud-main">
          <div class="ud-charts">
            <UserDashboardCharts
              v-model:startDate="startDate"
              v-model:endDate="endDate"
              v-model:granularity="granularity"
              :loading="loadingCharts"
              :trend="trendData"
              :models="modelStats"
              @dateRangeChange="loadCharts"
              @granularityChange="loadCharts"
              @refresh="refreshAll"
            />
          </div>

          <aside class="ud-side">
            <UserDashboardRecentUsage :data="recentUsage" :loading="loadingUsage" />
            <UserDashboardQuickActions />
          </aside>
        </div>

        <!-- 无数据：快捷操作横向 -->
        <UserDashboardQuickActions v-else variant="grid" />
      </template>
    </div>
  </AppLayout>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'
import { useAuthStore } from '@/stores/auth'
import { useAppStore } from '@/stores/app'
import { usageAPI, type UserDashboardStats as UserStatsType } from '@/api/usage'
import AppLayout from '@/components/layout/AppLayout.vue'
import DashboardSkeleton from '@/components/admin/dashboard/DashboardSkeleton.vue'
import DashboardErrorState from '@/components/admin/dashboard/DashboardErrorState.vue'
import DashboardGetStarted from '@/components/user/dashboard/DashboardGetStarted.vue'
import DashboardHero from '@/components/user/dashboard/DashboardHero.vue'
import UserDashboardStats from '@/components/user/dashboard/UserDashboardStats.vue'
import UserDashboardCharts from '@/components/user/dashboard/UserDashboardCharts.vue'
import UserDashboardRecentUsage from '@/components/user/dashboard/UserDashboardRecentUsage.vue'
import UserDashboardQuickActions from '@/components/user/dashboard/UserDashboardQuickActions.vue'
import type { UsageLog, TrendDataPoint, ModelStat, PlatformQuotaItem } from '@/types'
import { getMyPlatformQuotas } from '@/api/user'
import { formatDateLocalInput } from '@/utils/format'

const { t } = useI18n()
const router = useRouter()
const authStore = useAuthStore()
const appStore = useAppStore()

const user = computed(() => authStore.user)

const displayName = computed(() => {
  const u = authStore.user
  if (!u) return ''
  const email = (u.email || '').trim()
  const emailLocal = email.includes('@') ? email.split('@')[0] : email
  const name = (u.username || '').trim()
  if (name && name !== email) return name
  return emailLocal || name
})

const greeting = computed(() => {
  const h = new Date().getHours()
  if (h < 6) return t('dashboard.greetingNight')
  if (h < 12) return t('dashboard.greetingMorning')
  if (h < 18) return t('dashboard.greetingAfternoon')
  return t('dashboard.greetingEvening')
})

const stats = ref<UserStatsType | null>(null)
const loading = ref(false)
const loadingUsage = ref(false)
const loadingCharts = ref(false)
const loadError = ref<string | null>(null)
const trendData = ref<TrendDataPoint[]>([])
const modelStats = ref<ModelStat[]>([])
const recentUsage = ref<UsageLog[]>([])
const platformQuotas = ref<PlatformQuotaItem[] | null>(null)

const startDate = ref(formatDateLocalInput(new Date(Date.now() - 6 * 86400000)))
const endDate = ref(formatDateLocalInput(new Date()))
const granularity = ref('day')

const hasAnyUsage = computed(() => {
  const s = stats.value
  if (!s) return false
  return (
    (s.total_requests || 0) > 0 ||
    (s.total_tokens || 0) > 0 ||
    (s.today_requests || 0) > 0 ||
    (s.today_tokens || 0) > 0
  )
})

const loadStats = async () => {
  loading.value = true
  try {
    await authStore.refreshUser()
    stats.value = await usageAPI.getDashboardStats()
    loadError.value = null
  } catch (error) {
    console.error('Failed to load dashboard stats:', error)
    loadError.value = error instanceof Error ? error.message : t('dashboard.failedToLoad')
    if (!stats.value) appStore.showError(t('dashboard.failedToLoad'))
  } finally {
    loading.value = false
  }
}

const loadCharts = async () => {
  loadingCharts.value = true
  try {
    const res = await Promise.all([
      usageAPI.getDashboardTrend({
        start_date: startDate.value,
        end_date: endDate.value,
        granularity: granularity.value as any
      }),
      usageAPI.getDashboardModels({ start_date: startDate.value, end_date: endDate.value })
    ])
    trendData.value = res[0].trend || []
    modelStats.value = res[1].models || []
  } catch (error) {
    console.error('Failed to load charts:', error)
  } finally {
    loadingCharts.value = false
  }
}

const loadRecent = async () => {
  loadingUsage.value = true
  try {
    const res = await usageAPI.getByDateRange(startDate.value, endDate.value)
    recentUsage.value = res.items.slice(0, 5)
  } catch (error) {
    console.error('Failed to load recent usage:', error)
  } finally {
    loadingUsage.value = false
  }
}

const loadPlatformQuotas = async () => {
  try {
    const data = await getMyPlatformQuotas()
    platformQuotas.value = data.platform_quotas ?? []
  } catch (error) {
    console.warn('Failed to load platform quotas:', error)
    platformQuotas.value = []
  }
}

const refreshAll = () => {
  void loadStats()
  void loadCharts()
  void loadRecent()
  void loadPlatformQuotas()
}

onMounted(() => {
  refreshAll()
})
</script>

<style scoped>
.ud-page {
  display: flex;
  flex-direction: column;
  gap: 0.875rem;
}

/* 问候行：轻量，不抢 Hero 的视觉 */
.ud-greet {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: 0.5rem;
  padding: 0 0.25rem;
}

.ud-greet-title {
  font-size: 1.125rem;
  font-weight: 700;
  letter-spacing: -0.01em;
  color: rgb(15 23 42);
}

:root.dark .ud-greet-title {
  color: #ffffff;
}

.ud-greet-sub {
  margin-top: 0.0625rem;
  font-size: 0.75rem;
  color: rgb(148 163 184);
}

:root.dark .ud-greet-sub {
  color: rgb(100 116 139);
}

.ud-stale {
  border-radius: 9999px;
  padding: 0.125rem 0.5rem;
  font-size: 0.625rem;
  font-weight: 600;
  color: rgb(180 83 9);
  background: rgb(254 243 199);
}

:root.dark .ud-stale {
  color: rgb(252 211 77);
  background: rgb(120 53 15 / 0.4);
}

/* 图表 + 侧栏 */
.ud-main {
  display: grid;
  grid-template-columns: 1fr;
  gap: 0.875rem;
}

@media (min-width: 1280px) {
  .ud-main {
    grid-template-columns: minmax(0, 2fr) minmax(0, 1fr);
    align-items: start;
  }
}

.ud-charts {
  min-width: 0;
}

.ud-side {
  display: flex;
  min-width: 0;
  flex-direction: column;
  gap: 0.875rem;
}
</style>
