<template>
  <AppLayout>
    <div class="mx-auto max-w-4xl space-y-5">
      <!-- ============ 页面头部 ============ -->
      <UserPageHeader :title="t('redeem.title')" :description="t('redeem.description')" icon="gift" />

      <!-- ============ 账户概览 + 兑换表单 ============ -->
      <div class="grid grid-cols-1 gap-5 lg:grid-cols-5">
        <!-- 余额卡片（渐变主视觉） -->
        <section class="redeem-balance lg:col-span-2">
          <span class="redeem-balance-glow" aria-hidden="true"></span>

          <div class="relative">
            <div class="flex items-center justify-between">
              <span class="text-xs font-medium uppercase tracking-wider text-white/75">
                {{ t('redeem.currentBalance') }}
              </span>
              <span class="redeem-balance-badge">
                <Icon name="creditCard" size="sm" :stroke-width="2" />
              </span>
            </div>

            <p class="mt-3 flex items-baseline gap-1 text-white">
              <span class="text-2xl font-medium opacity-80">$</span>
              <span class="text-4xl font-bold tabular-nums tracking-tight">
                {{ formattedBalance }}
              </span>
            </p>

            <div class="mt-4 flex items-center gap-4 border-t border-white/20 pt-3">
              <div>
                <p class="text-[10px] uppercase tracking-wider text-white/65">
                  {{ t('redeem.concurrency') }}
                </p>
                <p class="text-base font-semibold tabular-nums text-white">
                  {{ user?.concurrency || 0 }}
                  <span class="text-xs font-normal text-white/70">{{ t('redeem.requests') }}</span>
                </p>
              </div>
            </div>
          </div>
        </section>

        <!-- 兑换表单 -->
        <section class="card p-5 lg:col-span-3">
          <div class="mb-4 flex items-center gap-2">
            <span class="flex h-8 w-8 items-center justify-center rounded-lg bg-primary-500/10 text-primary-600 dark:bg-primary-500/15 dark:text-primary-400">
              <Icon name="key" size="sm" :stroke-width="2" />
            </span>
            <div>
              <h2 class="text-sm font-semibold text-gray-900 dark:text-white">
                {{ t('redeem.redeemCodeLabel') }}
              </h2>
              <p class="text-xs text-gray-500 dark:text-dark-400">
                {{ t('redeem.redeemCodeHint') }}
              </p>
            </div>
          </div>

          <form class="space-y-4" @submit.prevent="handleRedeem">
            <div class="relative">
              <div class="pointer-events-none absolute inset-y-0 left-0 flex items-center pl-3.5">
                <Icon name="gift" size="md" class="text-gray-400 dark:text-dark-500" />
              </div>
              <input
                id="code"
                v-model="redeemCode"
                type="text"
                required
                autocomplete="off"
                spellcheck="false"
                :placeholder="t('redeem.redeemCodePlaceholder')"
                :disabled="submitting"
                class="redeem-input"
              />
            </div>

            <button
              type="submit"
              :disabled="!redeemCode.trim() || submitting"
              class="redeem-submit"
            >
              <LoadingSpinner v-if="submitting" size="sm" />
              <Icon v-else name="checkCircle" size="sm" :stroke-width="2" />
              {{ submitting ? t('redeem.redeeming') : t('redeem.redeemButton') }}
            </button>
          </form>

          <!-- 兑换结果 -->
          <Transition name="redeem-fade">
            <div
              v-if="redeemResult"
              class="mt-4 flex items-start gap-3 rounded-xl bg-emerald-50 p-3.5 ring-1 ring-emerald-200/70 dark:bg-emerald-900/20 dark:ring-emerald-800/50"
              role="status"
            >
              <span class="flex h-8 w-8 flex-shrink-0 items-center justify-center rounded-lg bg-emerald-500/15 text-emerald-600 dark:text-emerald-400">
                <Icon name="checkCircle" size="md" :stroke-width="2" />
              </span>
              <div class="min-w-0 flex-1 text-sm">
                <p class="font-semibold text-emerald-800 dark:text-emerald-300">
                  {{ t('redeem.redeemSuccess') }}
                </p>
                <p class="mt-0.5 text-xs text-emerald-700/90 dark:text-emerald-400/90">
                  {{ redeemResult.message }}
                </p>
                <div class="mt-2 flex flex-wrap gap-x-4 gap-y-1 text-xs font-medium text-emerald-800 dark:text-emerald-300">
                  <span v-if="redeemResult.type === 'balance'" class="tabular-nums">
                    {{ t('redeem.added') }}: +${{ redeemResult.value.toFixed(2) }}
                  </span>
                  <span v-else-if="redeemResult.type === 'concurrency'" class="tabular-nums">
                    {{ t('redeem.added') }}: +{{ redeemResult.value }} {{ t('redeem.concurrentRequests') }}
                  </span>
                  <span v-else-if="redeemResult.type === 'subscription'">
                    {{ t('redeem.subscriptionAssigned') }}
                    <template v-if="redeemResult.group_name"> · {{ redeemResult.group_name }}</template>
                    <template v-if="redeemResult.validity_days">
                      · {{ t('redeem.subscriptionDays', { days: redeemResult.validity_days }) }}
                    </template>
                  </span>
                  <span v-if="redeemResult.new_balance !== undefined" class="tabular-nums opacity-80">
                    {{ t('redeem.newBalance') }}: ${{ redeemResult.new_balance.toFixed(2) }}
                  </span>
                  <span v-if="redeemResult.new_concurrency !== undefined" class="tabular-nums opacity-80">
                    {{ t('redeem.newConcurrency') }}: {{ redeemResult.new_concurrency }}
                  </span>
                </div>
              </div>
            </div>
          </Transition>

          <!-- 兑换失败 -->
          <Transition name="redeem-fade">
            <div
              v-if="errorMessage"
              class="mt-4 flex items-start gap-3 rounded-xl bg-red-50 p-3.5 ring-1 ring-red-200/70 dark:bg-red-900/20 dark:ring-red-800/50"
              role="alert"
            >
              <span class="flex h-8 w-8 flex-shrink-0 items-center justify-center rounded-lg bg-red-500/15 text-red-600 dark:text-red-400">
                <Icon name="exclamationCircle" size="md" :stroke-width="2" />
              </span>
              <div class="min-w-0 flex-1 text-sm">
                <p class="font-semibold text-red-800 dark:text-red-300">
                  {{ t('redeem.redeemFailed') }}
                </p>
                <p class="mt-0.5 text-xs text-red-700/90 dark:text-red-400/90">
                  {{ errorMessage }}
                </p>
              </div>
            </div>
          </Transition>
        </section>
      </div>

      <!-- ============ 兑换历史 ============ -->
      <section class="card overflow-hidden">
        <div class="flex flex-wrap items-center justify-between gap-3 border-b border-gray-100 px-5 py-3.5 dark:border-dark-700/60">
          <div class="flex items-center gap-2">
            <h2 class="text-sm font-semibold text-gray-900 dark:text-white">
              {{ t('redeem.recentActivity') }}
            </h2>
            <span v-if="historyTotal > 0" class="redeem-count-pill">
              {{ historyTotal }}
            </span>
          </div>

          <div class="flex items-center gap-2">
            <label class="flex items-center gap-1.5 text-xs text-gray-500 dark:text-dark-400">
              {{ t('pagination.perPage') }}
              <select
                v-model="historyPageSize"
                class="redeem-page-size"
                :disabled="loadingHistory || submitting"
                @change="fetchHistory(1)"
              >
                <option v-for="size in [20, 50, 100]" :key="size" :value="size">{{ size }}</option>
              </select>
            </label>
          </div>
        </div>

        <!-- 加载中 -->
        <div v-if="loadingHistory" class="space-y-2 p-4">
          <div
            v-for="i in 4"
            :key="i"
            class="flex items-center gap-3 rounded-xl bg-gray-50/70 p-3 dark:bg-dark-800/40"
          >
            <div class="h-9 w-9 flex-shrink-0 animate-pulse rounded-lg bg-gray-200 dark:bg-dark-600/60"></div>
            <div class="flex-1 space-y-1.5">
              <div class="h-3 w-32 animate-pulse rounded bg-gray-200 dark:bg-dark-600/60"></div>
              <div class="h-2.5 w-20 animate-pulse rounded bg-gray-200/70 dark:bg-dark-600/40"></div>
            </div>
            <div class="h-4 w-16 animate-pulse rounded bg-gray-200 dark:bg-dark-600/60"></div>
          </div>
        </div>

        <!-- 历史列表 -->
        <ul v-else-if="history.length > 0" class="divide-y divide-gray-50 dark:divide-dark-700/40">
          <li
            v-for="item in history"
            :key="item.id"
            class="redeem-history-row"
          >
            <span class="redeem-history-icon" :class="historyTone(item).iconBg">
              <Icon :name="historyTone(item).icon" size="md" :stroke-width="2" />
            </span>

            <div class="min-w-0 flex-1">
              <p class="truncate text-sm font-medium text-gray-900 dark:text-white">
                {{ getHistoryItemTitle(item) }}
              </p>
              <p class="mt-0.5 flex flex-wrap items-center gap-x-2 text-xs text-gray-500 dark:text-dark-400">
                <span>{{ formatDateTime(item.used_at) }}</span>
                <span class="text-gray-300 dark:text-dark-600">·</span>
                <span v-if="!isAdminAdjustment(item.type)" class="font-mono text-[11px]">
                  {{ item.code.slice(0, 8) }}...
                </span>
                <span v-else>{{ t('redeem.adminAdjustment') }}</span>
                <span v-if="item.notes" class="truncate italic opacity-80" :title="item.notes">
                  {{ item.notes }}
                </span>
              </p>
            </div>

            <span class="shrink-0 text-sm font-semibold tabular-nums" :class="historyTone(item).value">
              {{ formatHistoryValue(item) }}
            </span>
          </li>
        </ul>

        <!-- 空态 -->
        <div v-else class="flex flex-col items-center justify-center gap-2 px-4 py-12 text-center">
          <span class="redeem-empty-icon">
            <Icon name="clock" size="lg" :stroke-width="1.6" />
          </span>
          <p class="text-sm font-medium text-gray-600 dark:text-dark-300">
            {{ t('redeem.historyWillAppear') }}
          </p>
        </div>

        <!-- 分页 -->
        <div
          v-if="historyTotal > historyPageSize"
          class="flex flex-wrap items-center justify-between gap-3 border-t border-gray-100 px-5 py-3 dark:border-dark-700/60"
        >
          <span class="text-xs text-gray-500 dark:text-dark-400">
            {{ t('common.total') }}: {{ historyTotal }}
          </span>
          <div class="flex items-center gap-2">
            <button
              class="redeem-page-btn"
              :disabled="loadingHistory || submitting || historyPage <= 1"
              @click="fetchHistory(historyPage - 1)"
            >
              {{ t('pagination.previous') }}
            </button>
            <span class="text-xs tabular-nums text-gray-600 dark:text-dark-300">
              {{ historyPage }} / {{ Math.ceil(historyTotal / historyPageSize) }}
            </span>
            <button
              class="redeem-page-btn"
              :disabled="loadingHistory || submitting || historyPage * historyPageSize >= historyTotal"
              @click="fetchHistory(historyPage + 1)"
            >
              {{ t('pagination.next') }}
            </button>
          </div>
        </div>
      </section>

      <!-- ============ 规则说明 ============ -->
      <section class="redeem-info">
        <div class="flex items-start gap-3">
          <span class="redeem-info-icon">
            <Icon name="infoCircle" size="md" :stroke-width="2" />
          </span>
          <div class="min-w-0 flex-1">
            <h3 class="text-sm font-semibold text-primary-800 dark:text-primary-300">
              {{ t('redeem.aboutCodes') }}
            </h3>
            <ul class="mt-1.5 space-y-1 text-xs text-primary-700/90 dark:text-primary-400/90">
              <li class="flex gap-1.5">
                <span class="select-none opacity-60">•</span>{{ t('redeem.codeRule1') }}
              </li>
              <li class="flex gap-1.5">
                <span class="select-none opacity-60">•</span>{{ t('redeem.codeRule2') }}
              </li>
              <li class="flex gap-1.5">
                <span class="select-none opacity-60">•</span>
                <span>
                  {{ t('redeem.codeRule3') }}
                  <span v-if="contactInfo" class="redeem-contact">{{ contactInfo }}</span>
                </span>
              </li>
              <li class="flex gap-1.5">
                <span class="select-none opacity-60">•</span>{{ t('redeem.codeRule4') }}
              </li>
            </ul>
          </div>
        </div>
      </section>
    </div>
  </AppLayout>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { useAuthStore } from '@/stores/auth'
import { useAppStore } from '@/stores/app'
import { useSubscriptionStore } from '@/stores/subscriptions'
import { redeemAPI, authAPI, type RedeemHistoryItem } from '@/api'
import AppLayout from '@/components/layout/AppLayout.vue'
import Icon from '@/components/icons/Icon.vue'
import LoadingSpinner from '@/components/common/LoadingSpinner.vue'
import UserPageHeader from '@/components/common/UserPageHeader.vue'
import { formatDateTime } from '@/utils/format'

const { t } = useI18n()
const authStore = useAuthStore()
const appStore = useAppStore()
const subscriptionStore = useSubscriptionStore()

const user = computed(() => authStore.user)

const formattedBalance = computed(() => {
  const value = Number(user.value?.balance)
  return Number.isFinite(value) ? value.toFixed(2) : '0.00'
})

const redeemCode = ref('')
const submitting = ref(false)
const redeemResult = ref<{
  message: string
  type: string
  value: number
  new_balance?: number
  new_concurrency?: number
  group_name?: string
  validity_days?: number
} | null>(null)
const errorMessage = ref('')

// History data
const history = ref<RedeemHistoryItem[]>([])
const loadingHistory = ref(false)
const historyPage = ref(1)
const historyPageSize = ref(20)
const historyTotal = ref(0)
let historyRequest = 0
let loadedHistoryPageSize = 20
const contactInfo = ref('')

// Helper functions for history display
const isBalanceType = (type: string) => {
  return type === 'balance' || type === 'admin_balance'
}

const isSubscriptionType = (type: string) => {
  return type === 'subscription'
}

const isAdminAdjustment = (type: string) => {
  return type === 'admin_balance' || type === 'admin_concurrency'
}

interface HistoryTone {
  icon: 'dollar' | 'badge' | 'bolt'
  iconBg: string
  value: string
}

/** 按类型与正负号给出图标和配色，避免模板里嵌套三层三元表达式 */
const historyTone = (item: RedeemHistoryItem): HistoryTone => {
  if (isBalanceType(item.type)) {
    return item.value >= 0
      ? {
          icon: 'dollar',
          iconBg: 'bg-emerald-500/10 text-emerald-600 dark:bg-emerald-500/15 dark:text-emerald-400',
          value: 'text-emerald-600 dark:text-emerald-400'
        }
      : {
          icon: 'dollar',
          iconBg: 'bg-red-500/10 text-red-600 dark:bg-red-500/15 dark:text-red-400',
          value: 'text-red-600 dark:text-red-400'
        }
  }
  if (isSubscriptionType(item.type)) {
    return {
      icon: 'badge',
      iconBg: 'bg-purple-500/10 text-purple-600 dark:bg-purple-500/15 dark:text-purple-400',
      value: 'text-purple-600 dark:text-purple-400'
    }
  }
  return item.value >= 0
    ? {
        icon: 'bolt',
        iconBg: 'bg-blue-500/10 text-blue-600 dark:bg-blue-500/15 dark:text-blue-400',
        value: 'text-blue-600 dark:text-blue-400'
      }
    : {
        icon: 'bolt',
        iconBg: 'bg-orange-500/10 text-orange-600 dark:bg-orange-500/15 dark:text-orange-400',
        value: 'text-orange-600 dark:text-orange-400'
      }
}

const getHistoryItemTitle = (item: RedeemHistoryItem) => {
  if (item.type === 'balance') {
    return t('redeem.balanceAddedRedeem')
  } else if (item.type === 'admin_balance') {
    return item.value >= 0 ? t('redeem.balanceAddedAdmin') : t('redeem.balanceDeductedAdmin')
  } else if (item.type === 'concurrency') {
    return t('redeem.concurrencyAddedRedeem')
  } else if (item.type === 'admin_concurrency') {
    return item.value >= 0 ? t('redeem.concurrencyAddedAdmin') : t('redeem.concurrencyReducedAdmin')
  } else if (item.type === 'subscription') {
    return t('redeem.subscriptionAssigned')
  }
  return t('common.unknown')
}

const formatHistoryValue = (item: RedeemHistoryItem) => {
  if (isBalanceType(item.type)) {
    const sign = item.value >= 0 ? '+' : ''
    return `${sign}$${item.value.toFixed(2)}`
  } else if (isSubscriptionType(item.type)) {
    const days = item.validity_days || Math.round(item.value)
    const groupName = item.group?.name || ''
    return groupName ? `${days}${t('redeem.days')} · ${groupName}` : `${days}${t('redeem.days')}`
  } else {
    const sign = item.value >= 0 ? '+' : ''
    return `${sign}${item.value} ${t('redeem.requests')}`
  }
}

const fetchHistory = async (page = 1) => {
  const request = ++historyRequest
  const pageSize = historyPageSize.value
  loadingHistory.value = true
  try {
    const result = await redeemAPI.getHistory(page, pageSize)
    if (request !== historyRequest) return
    history.value = result.items
    historyTotal.value = result.total
    historyPage.value = page
    historyPageSize.value = pageSize
    loadedHistoryPageSize = pageSize
  } catch (error) {
    if (request !== historyRequest) return
    historyPageSize.value = loadedHistoryPageSize
    appStore.showError(t('redeem.historyLoadFailed'))
    console.error('Failed to fetch history:', error)
  } finally {
    if (request === historyRequest) loadingHistory.value = false
  }
}

const handleRedeem = async () => {
  if (!redeemCode.value.trim()) {
    appStore.showError(t('redeem.pleaseEnterCode'))
    return
  }

  submitting.value = true
  errorMessage.value = ''
  redeemResult.value = null

  try {
    const result = await redeemAPI.redeem(redeemCode.value.trim())

    redeemResult.value = result

    // Refresh user data to get updated balance/concurrency
    try {
      await authStore.refreshUser()
    } catch (error) {
      console.error('Failed to refresh user after redeem:', error)
      appStore.showWarning(t('redeem.userRefreshFailed'))
    }

    // If subscription type, immediately refresh subscription status
    if (result.type === 'subscription') {
      try {
        await subscriptionStore.fetchActiveSubscriptions(true) // force refresh
      } catch (error) {
        console.error('Failed to refresh subscriptions after redeem:', error)
        appStore.showWarning(t('redeem.subscriptionRefreshFailed'))
      }
    }

    // Clear the input
    redeemCode.value = ''

    // Refresh history
    await fetchHistory()

    // Show success toast
    appStore.showSuccess(t('redeem.codeRedeemSuccess'))
  } catch (error: any) {
    errorMessage.value = error.response?.data?.detail || t('redeem.failedToRedeem')

    appStore.showError(t('redeem.redeemFailed'))
  } finally {
    submitting.value = false
  }
}

onMounted(async () => {
  fetchHistory()
  try {
    const settings = await authAPI.getPublicSettings()
    contactInfo.value = settings.contact_info || ''
  } catch (error) {
    console.error('Failed to load contact info:', error)
  }
})
</script>

<style scoped>
/* ============ 余额卡片 ============ */
.redeem-balance {
  position: relative;
  overflow: hidden;
  border-radius: 1rem;
  padding: 1.25rem;
  background: linear-gradient(135deg, #14b8a6 0%, #0d9488 55%, #0f766e 100%);
  box-shadow: 0 8px 24px rgba(13, 148, 136, 0.22);
}

.redeem-balance-glow {
  position: absolute;
  right: -3rem;
  bottom: -4rem;
  height: 12rem;
  width: 12rem;
  border-radius: 9999px;
  background: radial-gradient(circle, rgba(255, 255, 255, 0.22) 0%, transparent 70%);
  pointer-events: none;
}

.redeem-balance-badge {
  display: inline-flex;
  height: 1.75rem;
  width: 1.75rem;
  align-items: center;
  justify-content: center;
  border-radius: 0.5rem;
  color: rgb(255 255 255);
  background: rgb(255 255 255 / 0.2);
}

/* ============ 输入框 ============ */
.redeem-input {
  width: 100%;
  border-radius: 0.75rem;
  padding: 0.875rem 1rem 0.875rem 2.75rem;
  font-size: 1rem;
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
  letter-spacing: 0.04em;
  color: rgb(17 24 39);
  background: rgb(249 250 251);
  border: 1px solid rgb(229 231 235);
  transition: border-color 150ms ease, box-shadow 150ms ease, background-color 150ms ease;
}

.redeem-input::placeholder {
  font-family: system-ui, sans-serif;
  letter-spacing: normal;
  font-size: 0.875rem;
  color: rgb(156 163 175);
}

.redeem-input:focus {
  outline: none;
  background: rgb(255 255 255);
  border-color: rgb(20 184 166);
  box-shadow: 0 0 0 3px rgba(20, 184, 166, 0.15);
}

.redeem-input:disabled {
  opacity: 0.6;
}

:root.dark .redeem-input {
  color: rgb(255 255 255);
  background: rgb(15 23 42 / 0.6);
  border-color: rgb(51 65 85);
}

:root.dark .redeem-input:focus {
  background: rgb(15 23 42 / 0.9);
  border-color: rgb(45 212 191);
  box-shadow: 0 0 0 3px rgba(45 212 191, 0.18);
}

/* ============ 提交按钮 ============ */
.redeem-submit {
  display: inline-flex;
  width: 100%;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  border-radius: 0.75rem;
  padding: 0.75rem 1rem;
  font-size: 0.875rem;
  font-weight: 600;
  color: rgb(255 255 255);
  background: linear-gradient(135deg, #14b8a6 0%, #0d9488 100%);
  box-shadow: 0 4px 12px rgba(13, 148, 136, 0.25);
  transition: transform 150ms ease, box-shadow 150ms ease, opacity 150ms ease;
}

.redeem-submit:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 8px 20px rgba(13, 148, 136, 0.32);
}

.redeem-submit:active:not(:disabled) {
  transform: translateY(0);
}

.redeem-submit:disabled {
  opacity: 0.55;
  cursor: not-allowed;
  box-shadow: none;
}

/* ============ 历史列表 ============ */
.redeem-history-row {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem 1.25rem;
  transition: background-color 150ms ease;
}

.redeem-history-row:hover {
  background: rgb(249 250 251);
}

:root.dark .redeem-history-row:hover {
  background: rgb(30 41 59 / 0.4);
}

.redeem-history-icon {
  display: flex;
  height: 2.25rem;
  width: 2.25rem;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  border-radius: 0.625rem;
}

.redeem-count-pill {
  display: inline-flex;
  align-items: center;
  border-radius: 9999px;
  padding: 0.0625rem 0.5rem;
  font-size: 0.6875rem;
  font-weight: 600;
  font-variant-numeric: tabular-nums;
  color: rgb(15 118 110);
  background: rgb(240 253 250);
  @apply ring-1 ring-primary-500/20;
}

:root.dark .redeem-count-pill {
  color: rgb(94 234 212);
  background: rgb(19 78 74 / 0.4);
  --tw-ring-color: rgb(20 184 166 / 0.25);
}

.redeem-page-size {
  border-radius: 0.5rem;
  border: 1px solid rgb(229 231 235);
  background: rgb(255 255 255);
  padding: 0.1875rem 0.375rem;
  font-size: 0.75rem;
  color: rgb(55 65 81);
}

:root.dark .redeem-page-size {
  border-color: rgb(51 65 85);
  background: rgb(30 41 59);
  color: rgb(203 213 225);
}

.redeem-page-btn {
  border-radius: 0.5rem;
  padding: 0.3125rem 0.75rem;
  font-size: 0.75rem;
  font-weight: 500;
  color: rgb(55 65 81);
  background: rgb(243 244 246);
  transition: background-color 150ms ease;
}

.redeem-page-btn:hover:not(:disabled) {
  background: rgb(229 231 235);
}

:root.dark .redeem-page-btn {
  color: rgb(203 213 225);
  background: rgb(30 41 59);
}

:root.dark .redeem-page-btn:hover:not(:disabled) {
  background: rgb(51 65 85);
}

.redeem-page-btn:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}

/* ============ 空态 ============ */
.redeem-empty-icon {
  display: flex;
  height: 2.75rem;
  width: 2.75rem;
  align-items: center;
  justify-content: center;
  border-radius: 0.75rem;
  color: rgb(156 163 175);
  background: rgb(249 250 251);
  @apply ring-1 ring-gray-200/70;
}

:root.dark .redeem-empty-icon {
  background: rgb(30 41 59 / 0.5);
  color: rgb(100 116 139);
  --tw-ring-color: rgb(51 65 85 / 0.6);
}

/* ============ 规则说明 ============ */
.redeem-info {
  border-radius: 1rem;
  padding: 0.875rem 1rem;
  background: rgb(240 253 250);
  @apply ring-1 ring-primary-500/15;
}

:root.dark .redeem-info {
  background: rgb(19 78 74 / 0.25);
  --tw-ring-color: rgb(20 184 166 / 0.18);
}

.redeem-info-icon {
  display: flex;
  height: 2rem;
  width: 2rem;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  border-radius: 0.5rem;
  color: rgb(13 148 136);
  background: rgb(255 255 255 / 0.8);
}

:root.dark .redeem-info-icon {
  color: rgb(94 234 212);
  background: rgb(19 78 74 / 0.6);
}

.redeem-contact {
  display: inline-flex;
  align-items: center;
  margin-left: 0.375rem;
  border-radius: 0.375rem;
  padding: 0.0625rem 0.375rem;
  font-size: 0.6875rem;
  font-weight: 600;
  color: rgb(15 118 110);
  background: rgb(204 251 241);
}

:root.dark .redeem-contact {
  color: rgb(153 246 228);
  background: rgb(17 94 89 / 0.6);
}

/* ============ 过渡 ============ */
.redeem-fade-enter-active,
.redeem-fade-leave-active {
  transition: opacity 200ms ease, transform 200ms ease;
}

.redeem-fade-enter-from,
.redeem-fade-leave-to {
  opacity: 0;
  transform: translateY(-6px);
}
</style>
