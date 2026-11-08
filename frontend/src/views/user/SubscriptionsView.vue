<template>
  <AppLayout>
    <div class="space-y-5">
      <!-- ============ 页面头部 ============ -->
      <UserPageHeader
        :title="t('userSubscriptions.title')"
        :description="t('userSubscriptions.description')"
        icon="badge"
      >
        <template #actions>
          <button
            type="button"
            class="sub-refresh-btn"
            :disabled="loading"
            @click="loadSubscriptions"
          >
            <Icon name="refresh" size="sm" :class="loading && 'animate-spin'" />
            <span>{{ t('common.refresh') }}</span>
          </button>
        </template>
      </UserPageHeader>

      <!-- ============ 概览统计 ============ -->
      <div v-if="!loading && subscriptions.length > 0" class="grid grid-cols-2 gap-3 lg:grid-cols-4">
        <StatChip
          icon="badge"
          tone="teal"
          :label="t('userSubscriptions.title')"
          :value="subscriptions.length"
        />
        <StatChip
          icon="checkCircle"
          tone="emerald"
          :label="t('userSubscriptions.status.active')"
          :value="activeCount"
        />
        <StatChip
          icon="calendar"
          tone="amber"
          :label="t('userSubscriptions.expires')"
          :value="soonestExpiryLabel"
        />
        <StatChip
          icon="bolt"
          tone="blue"
          :label="t('userSubscriptions.unlimited')"
          :value="unlimitedCount"
        />
      </div>

      <!-- ============ 加载骨架 ============ -->
      <div v-if="loading" class="grid gap-5 lg:grid-cols-2">
        <div v-for="i in 2" :key="i" class="card overflow-hidden">
          <div class="flex items-center gap-3 border-b border-gray-100 p-4 dark:border-dark-700/50">
            <div class="h-2 w-2 animate-pulse rounded-full bg-gray-200 dark:bg-dark-600/60"></div>
            <div class="flex-1 space-y-1.5">
              <div class="h-3.5 w-32 animate-pulse rounded bg-gray-200 dark:bg-dark-600/60"></div>
              <div class="h-2.5 w-20 animate-pulse rounded bg-gray-200/70 dark:bg-dark-600/40"></div>
            </div>
            <div class="h-5 w-16 animate-pulse rounded-full bg-gray-200 dark:bg-dark-600/60"></div>
          </div>
          <div class="space-y-4 p-4">
            <div v-for="j in 3" :key="j" class="space-y-1.5">
              <div class="flex justify-between">
                <div class="h-2.5 w-14 animate-pulse rounded bg-gray-200/70 dark:bg-dark-600/40"></div>
                <div class="h-2.5 w-20 animate-pulse rounded bg-gray-200/70 dark:bg-dark-600/40"></div>
              </div>
              <div class="h-1.5 w-full animate-pulse rounded-full bg-gray-200/70 dark:bg-dark-600/40"></div>
            </div>
          </div>
        </div>
      </div>

      <!-- ============ 空态 ============ -->
      <div v-else-if="subscriptions.length === 0" class="sub-empty">
        <span class="sub-empty-glow" aria-hidden="true"></span>
        <span class="sub-empty-icon">
          <Icon name="creditCard" size="lg" :stroke-width="1.8" />
        </span>
        <h3 class="mt-4 text-base font-semibold text-gray-900 dark:text-white">
          {{ t('userSubscriptions.noActiveSubscriptions') }}
        </h3>
        <p class="mt-1 max-w-sm text-sm text-gray-500 dark:text-dark-400">
          {{ t('userSubscriptions.noActiveSubscriptionsDesc') }}
        </p>
        <button
          type="button"
          class="sub-primary-btn mt-5"
          @click="router.push({ path: '/purchase', query: { tab: 'subscription' } })"
        >
          <Icon name="plus" size="sm" :stroke-width="2" />
          {{ t('nav.buySubscription') }}
        </button>
      </div>

      <!-- ============ 订阅卡片 ============ -->
      <div v-else class="grid gap-5 lg:grid-cols-2">
        <article
          v-for="subscription in subscriptions"
          :key="subscription.id"
          class="sub-card"
          :class="subscription.status === 'active' ? 'sub-card-active' : 'sub-card-inactive'"
        >
          <!-- 顶部强调条 -->
          <span
            class="sub-card-accent"
            :class="platformAccentBarClass(subscription.group?.platform || '')"
            aria-hidden="true"
          ></span>

          <!-- 卡片头部 -->
          <header class="flex items-start justify-between gap-3 border-b border-gray-100/80 p-4 dark:border-dark-700/50">
            <div class="flex min-w-0 items-start gap-2.5">
              <span
                class="mt-1.5 h-2 w-2 shrink-0 rounded-full ring-2 ring-white dark:ring-dark-800"
                :class="platformAccentDotClass(subscription.group?.platform || '')"
              ></span>

              <div class="min-w-0">
                <div class="flex flex-wrap items-center gap-2">
                  <h3 class="truncate text-sm font-semibold text-gray-900 dark:text-white">
                    {{ subscription.group?.name || `Group #${subscription.group_id}` }}
                  </h3>
                  <span :class="['sub-platform-badge', platformBadgeClass(subscription.group?.platform || '')]">
                    {{ platformLabel(subscription.group?.platform || '') }}
                  </span>
                </div>

                <p
                  v-if="subscription.group?.description"
                  class="mt-0.5 truncate text-xs text-gray-500 dark:text-dark-400"
                >
                  {{ subscription.group.description }}
                </p>

                <div class="mt-1.5 flex flex-wrap gap-x-3 gap-y-1 text-[11px] text-gray-400 dark:text-dark-500">
                  <span class="tabular-nums">
                    {{ t('payment.planCard.rate') }}: ×{{ subscription.group?.rate_multiplier ?? 1 }}
                  </span>
                  <span
                    v-if="subscriptionHasPeakRate(subscription)"
                    class="text-amber-600 dark:text-amber-400"
                  >
                    {{ t('payment.planCard.peakRate') }}: {{ subscriptionPeakRateLabel(subscription) }}
                  </span>
                </div>
              </div>
            </div>

            <div class="flex shrink-0 flex-col items-end gap-2">
              <span class="sub-status-badge" :class="statusBadgeClass(subscription.status)">
                <span class="h-1.5 w-1.5 rounded-full" :class="statusDotClass(subscription.status)"></span>
                {{ t(`userSubscriptions.status.${subscription.status}`) }}
              </span>
              <button
                v-if="subscription.status === 'active'"
                type="button"
                class="sub-renew-btn"
                @click="router.push({ path: '/purchase', query: { tab: 'subscription', group: String(subscription.group_id) } })"
              >
                {{ t('payment.renewNow') }}
              </button>
            </div>
          </header>

          <!-- 用量区 -->
          <div class="space-y-4 p-4">
            <!-- 到期时间 -->
            <div class="flex items-center justify-between text-sm">
              <span class="flex items-center gap-1.5 text-gray-500 dark:text-dark-400">
                <Icon name="calendar" size="xs" class="opacity-70" />
                {{ t('userSubscriptions.expires') }}
              </span>
              <span v-if="subscription.expires_at" :class="getExpirationClass(subscription.expires_at)">
                {{ formatExpirationDate(subscription.expires_at) }}
              </span>
              <span v-else class="text-gray-600 dark:text-dark-300">
                {{ t('userSubscriptions.noExpiration') }}
              </span>
            </div>

            <!-- 日/周/月用量 -->
            <div
              v-for="quota in quotaRows(subscription)"
              :key="quota.key"
              class="space-y-1.5"
            >
              <div class="flex items-baseline justify-between">
                <span class="text-xs font-medium text-gray-600 dark:text-dark-300">
                  {{ quota.label }}
                </span>
                <span class="text-xs tabular-nums">
                  <span class="font-semibold text-gray-900 dark:text-white">
                    ${{ quota.used.toFixed(2) }}
                  </span>
                  <span class="text-gray-400 dark:text-dark-500"> / ${{ quota.limit.toFixed(2) }}</span>
                </span>
              </div>

              <div class="sub-progress-track" role="presentation">
                <span
                  class="sub-progress-fill"
                  :class="quota.barClass"
                  :style="{ width: quota.width }"
                ></span>
              </div>

              <p v-if="quota.hint" class="text-[11px] text-gray-400 dark:text-dark-500">
                {{ quota.hint }}
              </p>
            </div>

            <!-- 无限制 -->
            <div v-if="quotaRows(subscription).length === 0" class="sub-unlimited">
              <span class="text-3xl leading-none text-emerald-600 dark:text-emerald-400">∞</span>
              <div class="min-w-0">
                <p class="text-sm font-medium text-emerald-700 dark:text-emerald-300">
                  {{ t('userSubscriptions.unlimited') }}
                </p>
                <p class="text-xs text-emerald-600/75 dark:text-emerald-400/75">
                  {{ t('userSubscriptions.unlimitedDesc') }}
                </p>
              </div>
            </div>
          </div>
        </article>
      </div>
    </div>
  </AppLayout>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRouter } from 'vue-router'
import { useAppStore } from '@/stores/app'
import subscriptionsAPI from '@/api/subscriptions'
import type { UserSubscription } from '@/types'
import AppLayout from '@/components/layout/AppLayout.vue'
import Icon from '@/components/icons/Icon.vue'
import StatChip from '@/components/common/StatChip.vue'
import UserPageHeader from '@/components/common/UserPageHeader.vue'
import { formatDateTimeToMinute } from '@/utils/format'
import { hasPeakRate, formatPeakRateWindow, serverTimezoneLabel } from '@/utils/peak-rate'
import { platformBadgeClass, platformLabel } from '@/utils/platformColors'
import {
  getExpirationDateRelation,
  getRemainingDurationParts,
  isOneTimeDailyQuota,
  type RemainingDurationParts
} from '@/utils/subscriptionQuota'

function platformAccentDotClass(p: string): string {
  switch (p) {
    case 'anthropic': return 'bg-orange-500'
    case 'openai': return 'bg-emerald-500'
    case 'antigravity': return 'bg-purple-500'
    case 'gemini': return 'bg-blue-500'
    default: return 'bg-gray-400'
  }
}

function platformAccentBarClass(p: string): string {
  switch (p) {
    case 'anthropic': return 'bg-gradient-to-r from-orange-400 to-orange-600'
    case 'openai': return 'bg-gradient-to-r from-emerald-400 to-emerald-600'
    case 'antigravity': return 'bg-gradient-to-r from-purple-400 to-purple-600'
    case 'gemini': return 'bg-gradient-to-r from-blue-400 to-blue-600'
    default: return 'bg-gradient-to-r from-gray-300 to-gray-500'
  }
}

const { t } = useI18n()
const router = useRouter()
const appStore = useAppStore()

const subscriptions = ref<UserSubscription[]>([])
const loading = ref(true)

// ============ 概览统计 ============
const activeCount = computed(
  () => subscriptions.value.filter((s) => s.status === 'active').length
)

const unlimitedCount = computed(
  () =>
    subscriptions.value.filter(
      (s) =>
        !s.group?.daily_limit_usd && !s.group?.weekly_limit_usd && !s.group?.monthly_limit_usd
    ).length
)

/** 最近的到期时间，用于顶部统计条 */
const soonestExpiryLabel = computed(() => {
  const now = Date.now()
  const upcoming = subscriptions.value
    .filter((s) => s.expires_at)
    .map((s) => new Date(s.expires_at as string).getTime())
    .filter((ts) => Number.isFinite(ts) && ts > now)
    .sort((a, b) => a - b)

  if (upcoming.length === 0) return '—'
  const days = Math.ceil((upcoming[0] - now) / (1000 * 60 * 60 * 24))
  return `${days}d`
})

interface QuotaRow {
  key: string
  label: string
  used: number
  limit: number
  width: string
  barClass: string
  hint: string
}

/** 把日/周/月额度整理成统一结构，模板只负责渲染 */
function quotaRows(subscription: UserSubscription): QuotaRow[] {
  const rows: QuotaRow[] = []

  const dailyLimit = subscription.group?.daily_limit_usd
  if (dailyLimit) {
    rows.push({
      key: 'daily',
      label: t('userSubscriptions.daily'),
      used: subscription.daily_usage_usd || 0,
      limit: dailyLimit,
      width: getProgressWidth(subscription.daily_usage_usd, dailyLimit),
      barClass: getProgressBarClass(subscription.daily_usage_usd, dailyLimit),
      hint: subscription.daily_window_start ? formatDailyUsageWindow(subscription) : ''
    })
  }

  const weeklyLimit = subscription.group?.weekly_limit_usd
  if (weeklyLimit) {
    rows.push({
      key: 'weekly',
      label: t('userSubscriptions.weekly'),
      used: subscription.weekly_usage_usd || 0,
      limit: weeklyLimit,
      width: getProgressWidth(subscription.weekly_usage_usd, weeklyLimit),
      barClass: getProgressBarClass(subscription.weekly_usage_usd, weeklyLimit),
      hint: subscription.weekly_window_start
        ? t('userSubscriptions.resetIn', {
            time: formatResetTime(subscription.weekly_window_start, 168)
          })
        : ''
    })
  }

  const monthlyLimit = subscription.group?.monthly_limit_usd
  if (monthlyLimit) {
    rows.push({
      key: 'monthly',
      label: t('userSubscriptions.monthly'),
      used: subscription.monthly_usage_usd || 0,
      limit: monthlyLimit,
      width: getProgressWidth(subscription.monthly_usage_usd, monthlyLimit),
      barClass: getProgressBarClass(subscription.monthly_usage_usd, monthlyLimit),
      hint: subscription.monthly_window_start
        ? t('userSubscriptions.resetIn', {
            time: formatResetTime(subscription.monthly_window_start, 720)
          })
        : ''
    })
  }

  return rows
}

function statusBadgeClass(status: string): string {
  if (status === 'active') return 'sub-status-active'
  if (status === 'expired') return 'sub-status-expired'
  return 'sub-status-other'
}

function statusDotClass(status: string): string {
  if (status === 'active') return 'bg-emerald-500'
  if (status === 'expired') return 'bg-gray-400'
  return 'bg-red-500'
}

function subscriptionHasPeakRate(subscription: UserSubscription): boolean {
  return hasPeakRate(subscription.group)
}

function subscriptionPeakRateLabel(subscription: UserSubscription): string {
  return formatPeakRateWindow(subscription.group, serverTimezoneLabel(appStore.cachedPublicSettings?.server_utc_offset))
}

async function loadSubscriptions() {
  try {
    loading.value = true
    subscriptions.value = await subscriptionsAPI.getMySubscriptions()
  } catch (error) {
    console.error('Failed to load subscriptions:', error)
    appStore.showError(t('userSubscriptions.failedToLoad'))
  } finally {
    loading.value = false
  }
}

function getProgressWidth(used: number | undefined, limit: number | null | undefined): string {
  if (!limit || limit === 0) return '0%'
  const percentage = Math.min(((used || 0) / limit) * 100, 100)
  return `${percentage}%`
}

function getProgressBarClass(used: number | undefined, limit: number | null | undefined): string {
  if (!limit || limit === 0) return 'sub-bar-neutral'
  const percentage = ((used || 0) / limit) * 100
  if (percentage >= 90) return 'sub-bar-danger'
  if (percentage >= 70) return 'sub-bar-warn'
  return 'sub-bar-ok'
}

function formatExpirationDate(expiresAt: string): string {
  const now = new Date()
  const expires = new Date(expiresAt)
  const diff = expires.getTime() - now.getTime()
  const days = Math.ceil(diff / (1000 * 60 * 60 * 24))
  const relation = getExpirationDateRelation(expires, now)

  if (relation === null) return ''

  if (relation === 'expired') {
    return t('userSubscriptions.status.expired')
  }

  const dateStr = formatDateTimeToMinute(expires)

  if (relation === 'today') {
    return `${dateStr} (${t('common.today')})`
  }
  if (relation === 'tomorrow') {
    return `${dateStr} (${t('common.tomorrow')})`
  }

  return t('userSubscriptions.daysRemaining', { days }) + ` (${dateStr})`
}

function getExpirationClass(expiresAt: string): string {
  const now = new Date()
  const expires = new Date(expiresAt)
  const diff = expires.getTime() - now.getTime()
  const days = Math.ceil(diff / (1000 * 60 * 60 * 24))

  if (diff <= 0) return 'font-medium text-red-600 dark:text-red-400'
  if (days <= 3) return 'text-red-600 dark:text-red-400'
  if (days <= 7) return 'text-orange-600 dark:text-orange-400'
  return 'text-gray-700 dark:text-dark-200'
}

function formatDurationParts(parts: RemainingDurationParts): string {
  if (parts.days > 0) {
    return `${parts.days}d ${parts.hours}h`
  }

  if (parts.hours > 0) {
    return `${parts.hours}h ${parts.minutes}m`
  }

  return `${parts.minutes}m`
}

function formatDailyUsageWindow(subscription: UserSubscription): string {
  if (isOneTimeDailyQuota(subscription) && subscription.expires_at) {
    const parts = getRemainingDurationParts(subscription.expires_at)
    if (!parts) return t('userSubscriptions.windowNotActive')
    return t('userSubscriptions.quotaEndsIn', { time: formatDurationParts(parts) })
  }

  return t('userSubscriptions.resetIn', {
    time: formatResetTime(subscription.daily_window_start, 24)
  })
}

function formatResetTime(windowStart: string | null, windowHours: number): string {
  if (!windowStart) return t('userSubscriptions.windowNotActive')

  const start = new Date(windowStart)
  const end = new Date(start.getTime() + windowHours * 60 * 60 * 1000)
  const parts = getRemainingDurationParts(end)

  return parts ? formatDurationParts(parts) : t('userSubscriptions.windowNotActive')
}

onMounted(() => {
  loadSubscriptions()
})
</script>

<style scoped>
/* ============ 订阅卡片 ============ */
.sub-card {
  position: relative;
  overflow: hidden;
  border-radius: 1rem;
  background: rgb(255 255 255);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.04), 0 1px 2px rgba(0, 0, 0, 0.06);
  transition: transform 250ms ease, box-shadow 250ms ease;
  @apply ring-1 ring-gray-100;
}

:root.dark .sub-card {
  background: rgb(30 41 59 / 0.5);
  --tw-ring-color: rgb(51 65 85 / 0.5);
}

.sub-card-active:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
}

.sub-card-inactive {
  opacity: 0.82;
}

.sub-card-accent {
  position: absolute;
  inset: 0 0 auto 0;
  height: 3px;
}

.sub-platform-badge {
  border-radius: 0.375rem;
  border-width: 1px;
  padding: 0.0625rem 0.375rem;
  font-size: 0.625rem;
  font-weight: 600;
}

/* ============ 状态徽标 ============ */
.sub-status-badge {
  display: inline-flex;
  align-items: center;
  gap: 0.3125rem;
  border-radius: 9999px;
  padding: 0.125rem 0.5rem;
  font-size: 0.6875rem;
  font-weight: 600;
}

.sub-status-active {
  color: rgb(4 120 87);
  background: rgb(209 250 229);
}

:root.dark .sub-status-active {
  color: rgb(110 231 183);
  background: rgb(6 78 59 / 0.5);
}

.sub-status-expired {
  color: rgb(75 85 99);
  background: rgb(243 244 246);
}

:root.dark .sub-status-expired {
  color: rgb(156 163 175);
  background: rgb(51 65 85 / 0.6);
}

.sub-status-other {
  color: rgb(185 28 28);
  background: rgb(254 226 226);
}

:root.dark .sub-status-other {
  color: rgb(252 165 165);
  background: rgb(127 29 29 / 0.4);
}

/* ============ 续费按钮 ============ */
.sub-renew-btn {
  border-radius: 0.5rem;
  padding: 0.3125rem 0.75rem;
  font-size: 0.75rem;
  font-weight: 600;
  color: rgb(255 255 255);
  background: linear-gradient(135deg, #14b8a6 0%, #0d9488 100%);
  box-shadow: 0 2px 6px rgba(13, 148, 136, 0.22);
  transition: transform 150ms ease, box-shadow 150ms ease;
}

.sub-renew-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(13, 148, 136, 0.3);
}

/* ============ 进度条 ============ */
.sub-progress-track {
  position: relative;
  height: 0.375rem;
  overflow: hidden;
  border-radius: 9999px;
  background: rgb(243 244 246);
}

:root.dark .sub-progress-track {
  background: rgb(51 65 85 / 0.6);
}

.sub-progress-fill {
  display: block;
  height: 100%;
  border-radius: 9999px;
  transition: width 700ms cubic-bezier(0.16, 1, 0.3, 1);
}

.sub-bar-ok {
  background: linear-gradient(90deg, #34d399 0%, #059669 100%);
}

.sub-bar-warn {
  background: linear-gradient(90deg, #fbbf24 0%, #ea580c 100%);
}

.sub-bar-danger {
  background: linear-gradient(90deg, #f87171 0%, #dc2626 100%);
}

.sub-bar-neutral {
  background: rgb(156 163 175);
}

/* ============ 无限制 ============ */
.sub-unlimited {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  border-radius: 0.75rem;
  padding: 0.875rem 1rem;
  background: linear-gradient(90deg, rgb(236 253 245) 0%, rgb(240 253 250) 100%);
  @apply ring-1 ring-emerald-200/70;
}

:root.dark .sub-unlimited {
  background: linear-gradient(90deg, rgb(6 78 59 / 0.3) 0%, rgb(19 78 74 / 0.25) 100%);
  --tw-ring-color: rgb(6 95 70 / 0.5);
}

/* ============ 空态 ============ */
.sub-empty {
  position: relative;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  align-items: center;
  border-radius: 1rem;
  padding: 3.5rem 1.5rem;
  text-align: center;
  background: rgb(255 255 255);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.04);
  @apply ring-1 ring-gray-100;
}

:root.dark .sub-empty {
  background: rgb(30 41 59 / 0.5);
  --tw-ring-color: rgb(51 65 85 / 0.5);
}

.sub-empty-glow {
  position: absolute;
  top: -5rem;
  left: 50%;
  height: 12rem;
  width: 24rem;
  transform: translateX(-50%);
  border-radius: 9999px;
  background: radial-gradient(circle, rgb(20 184 166 / 0.12) 0%, transparent 70%);
  pointer-events: none;
}

.sub-empty-icon {
  position: relative;
  display: flex;
  height: 3.5rem;
  width: 3.5rem;
  align-items: center;
  justify-content: center;
  border-radius: 0.875rem;
  color: rgb(13 148 136);
  background: rgb(240 253 250);
  @apply ring-1 ring-primary-500/15;
}

:root.dark .sub-empty-icon {
  color: rgb(94 234 212);
  background: rgb(19 78 74 / 0.4);
  --tw-ring-color: rgb(20 184 166 / 0.2);
}

/* ============ 按钮 ============ */
.sub-primary-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  border-radius: 0.625rem;
  padding: 0.5rem 0.875rem;
  font-size: 0.8125rem;
  font-weight: 600;
  color: rgb(255 255 255);
  background: linear-gradient(135deg, #14b8a6 0%, #0d9488 100%);
  box-shadow: 0 4px 12px rgba(13, 148, 136, 0.25);
  transition: transform 150ms ease, box-shadow 150ms ease;
}

.sub-primary-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 8px 20px rgba(13, 148, 136, 0.32);
}

.sub-refresh-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  border-radius: 0.625rem;
  padding: 0.4375rem 0.8125rem;
  font-size: 0.75rem;
  font-weight: 500;
  color: rgb(15 118 110);
  background: rgb(255 255 255 / 0.9);
  transition: background-color 150ms ease, box-shadow 150ms ease;
  @apply ring-1 ring-primary-500/20;
}

.sub-refresh-btn:hover:not(:disabled) {
  background: rgb(255 255 255);
  box-shadow: 0 0 20px rgba(20, 184, 166, 0.25);
}

:root.dark .sub-refresh-btn {
  color: rgb(94 234 212);
  background: rgb(30 41 59 / 0.7);
  --tw-ring-color: rgb(20 184 166 / 0.25);
}

.sub-refresh-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
</style>
