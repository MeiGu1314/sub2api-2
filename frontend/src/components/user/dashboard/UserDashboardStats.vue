<template>
  <!--
    统计明细带：仅在 Hero 下方补充"累计/明细"信息。
    主视觉已在 Hero 承担，这里刻意保持低存在感（细分隔条，不是卡片墙）。
  -->
  <div class="sb-root">
    <div class="sb-band">
      <div class="sb-cell">
        <span class="sb-label">{{ t('dashboard.totalRequests') }}</span>
        <span class="sb-value">{{ formatNumber(stats?.total_requests || 0) }}</span>
        <span class="sb-sub">
          {{ t('dashboard.todayShort') }} {{ formatNumber(stats?.today_requests || 0) }}
        </span>
      </div>

      <div class="sb-cell">
        <span class="sb-label">{{ t('dashboard.totalTokens') }}</span>
        <span class="sb-value">{{ formatTokens(stats?.total_tokens || 0) }}</span>
        <span class="sb-sub">
          {{ t('dashboard.todayShort') }} {{ formatTokens(stats?.today_tokens || 0) }}
        </span>
      </div>

      <div class="sb-cell">
        <span class="sb-label">{{ t('dashboard.totalCost') }}</span>
        <span class="sb-value">${{ formatCost(stats?.total_actual_cost || 0) }}</span>
        <span class="sb-sub">
          {{ t('dashboard.standard') }} ${{ formatCost(stats?.total_cost || 0) }}
        </span>
      </div>

      <div class="sb-cell">
        <span class="sb-label">{{ t('dashboard.apiKeys') }}</span>
        <span class="sb-value">{{ stats?.total_api_keys || 0 }}</span>
        <span class="sb-sub">
          {{ stats?.active_api_keys || 0 }} {{ t('common.active') }}
        </span>
      </div>

      <div class="sb-cell">
        <span class="sb-label">{{ t('dashboard.avgResponse') }}</span>
        <span class="sb-value">{{ formatDuration(stats?.average_duration_ms || 0) }}</span>
        <span class="sb-sub">{{ formatTokens(stats?.tpm || 0) }} TPM</span>
      </div>
    </div>

    <!-- Token 构成条 -->
    <div v-if="totalTokens > 0" class="sb-compose">
      <div class="sb-compose-head">
        <span class="sb-compose-title">{{ t('dashboard.tokenComposition') }}</span>
        <span class="sb-compose-total">{{ formatTokens(totalTokens) }}</span>
      </div>

      <div class="sb-bar" role="presentation">
        <span class="sb-seg sb-seg-input" :style="{ width: `${pct.input}%` }"></span>
        <span class="sb-seg sb-seg-output" :style="{ width: `${pct.output}%` }"></span>
        <span class="sb-seg sb-seg-cache" :style="{ width: `${pct.cache}%` }"></span>
      </div>

      <div class="sb-legend">
        <span class="sb-legend-item">
          <i class="sb-dot sb-seg-input"></i>
          {{ t('dashboard.input') }}
          <b>{{ formatTokens(stats?.total_input_tokens || 0) }}</b>
          <em>{{ pct.input.toFixed(0) }}%</em>
        </span>
        <span class="sb-legend-item">
          <i class="sb-dot sb-seg-output"></i>
          {{ t('dashboard.output') }}
          <b>{{ formatTokens(stats?.total_output_tokens || 0) }}</b>
          <em>{{ pct.output.toFixed(0) }}%</em>
        </span>
        <span class="sb-legend-item">
          <i class="sb-dot sb-seg-cache"></i>
          {{ t('dashboard.cache') }}
          <b>{{ formatTokens(cacheTokens) }}</b>
          <em>{{ pct.cache.toFixed(0) }}%</em>
        </span>
      </div>
    </div>

    <!-- 平台 -->
    <div v-if="!isSimple && platformCards.length > 0" class="sb-platforms">
      <div class="mb-2.5 flex items-center justify-between">
        <span class="sb-compose-title">{{ t('dashboard.platformBreakdown') }}</span>
        <span class="sb-count">{{ t('dashboard.platformCount', { count: platformCount }) }}</span>
      </div>

      <div class="grid grid-cols-1 gap-2 sm:grid-cols-2 lg:grid-cols-4">
        <div
          v-for="item in platformCards"
          :key="item.platform"
          data-testid="platform-card"
          :data-platform="item.platform"
          class="sb-platform"
          :class="{ 'sb-platform-other': item.isOther }"
        >
          <div class="flex items-center justify-between gap-2">
            <span class="truncate text-xs font-semibold text-gray-900 dark:text-white">
              {{ item.isOther ? t('dashboard.platformOther') : platformLabel(item.platform) }}
            </span>
            <span class="shrink-0 font-mono text-xs font-semibold text-primary-600 dark:text-primary-400">
              ${{ formatCost(item.total_actual_cost) }}
            </span>
          </div>

          <div class="mt-2 flex items-center gap-3 text-[11px] text-gray-500 dark:text-dark-400">
            <span>{{ formatNumber(item.total_requests) }} {{ t('dashboard.requestsShort') }}</span>
            <span class="tabular-nums">{{ formatTokens(item.total_tokens) }}</span>
          </div>

          <!-- 配额 -->
          <div v-if="hasAnyLimit(item.quota) && !item.isOther" class="mt-2 space-y-1">
            <p class="text-[9px] font-semibold uppercase tracking-wide text-gray-400">
              {{ t('dashboard.platformQuota.title') }}
            </p>
            <template v-for="w in (['daily', 'weekly', 'monthly'] as const)" :key="w">
              <div v-if="quotaVal(item.quota, `${w}_limit_usd`) != null">
                <div class="flex items-center justify-between text-[10px]">
                  <span class="text-gray-500 dark:text-dark-400">
                    {{ t(`dashboard.platformQuota.${w}`) }}
                  </span>
                  <span class="font-mono text-gray-600 dark:text-gray-300">
                    <template v-if="(quotaVal(item.quota, `${w}_limit_usd`) as number) === 0">
                      {{ t('dashboard.platformQuota.disabled') }}
                    </template>
                    <template v-else>
                      ${{ formatUsd((quotaVal(item.quota, `${w}_usage_usd`) as number) ?? 0) }}/${{
                        formatUsd(quotaVal(item.quota, `${w}_limit_usd`) as number)
                      }}
                    </template>
                  </span>
                </div>
                <div class="sb-quota" role="presentation">
                  <div
                    class="sb-quota-fill"
                    :class="
                      quotaBarClass(
                        calcPercent(
                          (quotaVal(item.quota, `${w}_usage_usd`) as number) ?? 0,
                          quotaVal(item.quota, `${w}_limit_usd`) as number
                        )
                      )
                    "
                    :style="{
                      width:
                        calcPercent(
                          (quotaVal(item.quota, `${w}_usage_usd`) as number) ?? 0,
                          quotaVal(item.quota, `${w}_limit_usd`) as number
                        ) + '%'
                    }"
                  />
                </div>
              </div>
            </template>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import Icon from '@/components/icons/Icon.vue'
import type { PlatformDashboardStats, UserDashboardStats as UserStatsType } from '@/api/usage'
import type { PlatformQuotaItem } from '@/types'

void Icon

interface FusedPlatformCard {
  platform: string
  total_actual_cost: number
  today_actual_cost: number
  total_requests: number
  total_tokens: number
  isOther?: boolean
  quota?: PlatformQuotaItem
}

const props = defineProps<{
  stats: UserStatsType
  balance: number
  isSimple: boolean
  platformQuotas?: PlatformQuotaItem[] | null
}>()

defineEmits<{ (e: 'navigate', path: string): void }>()

const { t } = useI18n()

const totalTokens = computed(() => Number(props.stats?.total_tokens) || 0)
const cacheTokens = computed(
  () =>
    (Number(props.stats?.total_cache_creation_tokens) || 0) +
    (Number(props.stats?.total_cache_read_tokens) || 0)
)

/** Token 构成百分比 */
const pct = computed(() => {
  const total = totalTokens.value || 1
  const input = Number(props.stats?.total_input_tokens) || 0
  const output = Number(props.stats?.total_output_tokens) || 0
  const cache = cacheTokens.value
  return {
    input: (input / total) * 100,
    output: (output / total) * 100,
    cache: (cache / total) * 100
  }
})

const PLATFORM_LABELS: Record<string, string> = {
  anthropic: 'Claude',
  openai: 'OpenAI',
  gemini: 'Gemini',
  antigravity: 'Antigravity',
  grok: 'Grok',
  kimi: 'Kimi',
  zhipu: 'Zhipu GLM',
  deepseek: 'DeepSeek',
  minimax: 'MiniMax',
  typesafe: 'TypeSafe / Jev'
}

const platformLabel = (p: string) => PLATFORM_LABELS[p] ?? p

const OTHER_THRESHOLD = 0.0001
const platformCards = computed<FusedPlatformCard[]>(() => {
  const byPlat = new Map<string, PlatformDashboardStats>()
  for (const item of props.stats?.by_platform ?? []) byPlat.set(item.platform, item)

  const byQuota = new Map<string, PlatformQuotaItem>()
  for (const q of props.platformQuotas ?? []) byQuota.set(q.platform, q)

  const platforms = new Set<string>(byPlat.keys())
  for (const [platform, q] of byQuota) {
    if (hasAnyLimit(q)) platforms.add(platform)
  }

  const PLATFORM_ORDER = ['anthropic', 'openai', 'gemini', 'antigravity', 'grok', 'typesafe']
  const cards: FusedPlatformCard[] = []

  for (const p of platforms) {
    const stat = byPlat.get(p)
    cards.push({
      platform: p,
      total_actual_cost: stat?.total_actual_cost ?? 0,
      today_actual_cost: stat?.today_actual_cost ?? 0,
      total_requests: stat?.total_requests ?? 0,
      total_tokens: stat?.total_tokens ?? 0,
      quota: byQuota.get(p)
    })
  }

  cards.sort((a, b) => {
    const ai = PLATFORM_ORDER.indexOf(a.platform)
    const bi = PLATFORM_ORDER.indexOf(b.platform)
    if (ai === -1 && bi === -1) return a.platform.localeCompare(b.platform)
    if (ai === -1) return 1
    if (bi === -1) return -1
    return ai - bi
  })

  const total = props.stats?.total_actual_cost ?? 0
  const today = props.stats?.today_actual_cost ?? 0
  const sumTotal = cards.reduce((s, c) => s + c.total_actual_cost, 0)
  const sumToday = cards.reduce((s, c) => s + c.today_actual_cost, 0)
  const diffTotal = Math.max(0, total - sumTotal)
  const diffToday = Math.max(0, today - sumToday)

  if (diffTotal > OTHER_THRESHOLD || diffToday > OTHER_THRESHOLD) {
    cards.push({
      platform: '__other__',
      total_actual_cost: diffTotal,
      today_actual_cost: diffToday,
      total_requests: 0,
      total_tokens: 0,
      isOther: true
    })
  }

  return cards
})

const platformCount = computed(() => platformCards.value.filter((c) => !c.isOther).length)

type QuotaWindow = 'daily' | 'weekly' | 'monthly'
type QuotaField =
  | `${QuotaWindow}_limit_usd`
  | `${QuotaWindow}_usage_usd`
  | `${QuotaWindow}_window_resets_at`

function quotaVal(q: PlatformQuotaItem | undefined, key: QuotaField): PlatformQuotaItem[QuotaField] {
  return q?.[key]
}

function hasAnyLimit(q: PlatformQuotaItem | undefined): boolean {
  if (!q) return false
  return q.daily_limit_usd != null || q.weekly_limit_usd != null || q.monthly_limit_usd != null
}

function calcPercent(usage: number, limit: number): number {
  if (!limit || limit <= 0) return 0
  return Math.min(100, Math.max(0, Math.round((usage / limit) * 100)))
}

function quotaBarClass(p: number): string {
  if (p >= 95) return 'sb-quota-red'
  if (p >= 75) return 'sb-quota-amber'
  return 'sb-quota-green'
}

const usdFormatter = new Intl.NumberFormat('en-US', {
  minimumFractionDigits: 2,
  maximumFractionDigits: 2
})
function formatUsd(n: number): string {
  if (!Number.isFinite(n)) return '0.00'
  return usdFormatter.format(n)
}

const formatNumber = (n: number) => (Number(n) || 0).toLocaleString()
const formatCost = (c: number) => {
  const n = Number(c) || 0
  return n.toFixed(n >= 1 ? 2 : 4)
}
const formatTokens = (v: number) => {
  const t = Number(v) || 0
  if (t >= 1_000_000) return `${(t / 1_000_000).toFixed(1)}M`
  if (t >= 1000) return `${(t / 1000).toFixed(1)}K`
  return t.toString()
}
const formatDuration = (ms: number) =>
  ms >= 1000 ? `${(ms / 1000).toFixed(2)}s` : `${(ms || 0).toFixed(0)}ms`
</script>

<style scoped>
.sb-root {
  border-radius: 1rem;
  padding: 0.875rem 1rem;
  background: rgb(255 255 255 / 0.7);
  border: 1px solid rgb(255 255 255 / 0.7);
  box-shadow: 0 1px 2px rgba(15, 23, 42, 0.04);
  backdrop-filter: blur(14px) saturate(1.15);
  -webkit-backdrop-filter: blur(14px) saturate(1.15);
}

:root.dark .sb-root {
  background: rgb(30 41 59 / 0.5);
  border-color: rgb(148 163 184 / 0.12);
}

/* ============ 指标带 ============ */
.sb-band {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 0.75rem 1rem;
}

@media (min-width: 768px) {
  .sb-band {
    grid-template-columns: repeat(5, minmax(0, 1fr));
  }
}

.sb-cell {
  display: flex;
  flex-direction: column;
  gap: 0.0625rem;
  min-width: 0;
}

.sb-label {
  font-size: 0.5625rem;
  font-weight: 600;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: rgb(148 163 184);
}

.sb-value {
  font-size: 1rem;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  color: rgb(15 23 42);
}

:root.dark .sb-value {
  color: #ffffff;
}

.sb-sub {
  font-size: 0.5625rem;
  color: rgb(148 163 184);
}

:root.dark .sb-sub {
  color: rgb(100 116 139);
}

/* ============ Token 构成 ============ */
.sb-compose {
  margin-top: 0.875rem;
  padding-top: 0.75rem;
  border-top: 1px solid rgb(241 245 249);
}

:root.dark .sb-compose {
  border-top-color: rgb(51 65 85 / 0.6);
}

.sb-compose-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0.375rem;
}

.sb-compose-title {
  font-size: 0.625rem;
  font-weight: 700;
  letter-spacing: 0.07em;
  text-transform: uppercase;
  color: rgb(148 163 184);
}

.sb-compose-total {
  font-size: 0.6875rem;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  color: rgb(15 23 42);
}

:root.dark .sb-compose-total {
  color: #ffffff;
}

.sb-bar {
  display: flex;
  height: 0.375rem;
  overflow: hidden;
  border-radius: 9999px;
  background: rgb(241 245 249);
}

:root.dark .sb-bar {
  background: rgb(51 65 85 / 0.7);
}

.sb-seg {
  display: block;
  height: 100%;
  transition: width 700ms cubic-bezier(0.16, 1, 0.3, 1);
}

.sb-seg-input {
  background: linear-gradient(90deg, #38bdf8, #0284c7);
}

.sb-seg-output {
  background: linear-gradient(90deg, #a78bfa, #7c3aed);
}

.sb-seg-cache {
  background: linear-gradient(90deg, #34d399, #059669);
}

.sb-legend {
  display: flex;
  flex-wrap: wrap;
  gap: 0.25rem 1rem;
  margin-top: 0.4375rem;
}

.sb-legend-item {
  display: inline-flex;
  align-items: center;
  gap: 0.3125rem;
  font-size: 0.625rem;
  color: rgb(100 116 139);
}

:root.dark .sb-legend-item {
  color: rgb(148 163 184);
}

.sb-legend-item b {
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  color: rgb(15 23 42);
}

:root.dark .sb-legend-item b {
  color: #ffffff;
}

.sb-legend-item em {
  font-style: normal;
  font-weight: 600;
  opacity: 0.6;
}

.sb-dot {
  display: inline-block;
  height: 0.375rem;
  width: 0.375rem;
  border-radius: 9999px;
}

/* ============ 平台 ============ */
.sb-platforms {
  margin-top: 0.875rem;
  padding-top: 0.75rem;
  border-top: 1px solid rgb(241 245 249);
}

:root.dark .sb-platforms {
  border-top-color: rgb(51 65 85 / 0.6);
}

.sb-count {
  border-radius: 9999px;
  padding: 0.0625rem 0.4375rem;
  font-size: 0.5625rem;
  font-weight: 600;
  color: rgb(15 118 110);
  background: rgb(240 253 250);
}

:root.dark .sb-count {
  color: rgb(94 234 212);
  background: rgb(19 78 74 / 0.45);
}

.sb-platform {
  border-radius: 0.625rem;
  padding: 0.625rem 0.6875rem;
  background: rgb(248 250 252 / 0.9);
  border: 1px solid rgb(226 232 240 / 0.9);
  transition: border-color 160ms ease;
}

.sb-platform:hover {
  border-color: rgb(20 184 166 / 0.35);
}

:root.dark .sb-platform {
  background: rgb(15 23 42 / 0.4);
  border-color: rgb(51 65 85 / 0.7);
}

.sb-platform-other {
  border-style: dashed;
}

.sb-quota {
  height: 0.1875rem;
  width: 100%;
  overflow: hidden;
  border-radius: 9999px;
  background: rgb(226 232 240);
}

:root.dark .sb-quota {
  background: rgb(51 65 85);
}

.sb-quota-fill {
  height: 100%;
  border-radius: 9999px;
  transition: width 600ms cubic-bezier(0.16, 1, 0.3, 1);
}

.sb-quota-green {
  background: linear-gradient(90deg, #34d399, #059669);
}

.sb-quota-amber {
  background: linear-gradient(90deg, #fbbf24, #d97706);
}

.sb-quota-red {
  background: linear-gradient(90deg, #fb7185, #e11d48);
}
</style>
