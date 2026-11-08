<template>
  <!--
    用户端仪表盘主视觉：
    左侧大号余额，右侧今日关键数字，底部一行微型指标。
    所有数值由父组件传入，纯展示。
  -->
  <section class="oh">
    <span class="oh-glow" aria-hidden="true"></span>
    <span class="oh-ring" aria-hidden="true"></span>

    <div class="relative grid grid-cols-1 gap-6 lg:grid-cols-[minmax(0,1fr)_auto] lg:items-center">
      <!-- 左：余额 -->
      <div class="min-w-0">
        <p class="oh-eyebrow">
          <span class="oh-dot" aria-hidden="true"></span>
          {{ t('dashboard.balance') }}
          <span v-if="isSimple" class="oh-simple">{{ t('dashboard.simpleMode') }}</span>
        </p>

        <p class="oh-amount">
          <span class="oh-currency">$</span>
          <span class="oh-int">{{ balanceParts.int }}</span>
          <span class="oh-frac">.{{ balanceParts.frac }}</span>
        </p>

        <div class="mt-4 flex flex-wrap items-center gap-2">
          <button type="button" class="oh-cta" @click="$emit('navigate', '/redeem')">
            <Icon name="gift" size="sm" :stroke-width="2.2" />
            {{ t('dashboard.redeemCode') }}
          </button>
          <button type="button" class="oh-cta-ghost" @click="$emit('navigate', '/keys')">
            <Icon name="key" size="sm" :stroke-width="2" />
            {{ t('dashboard.createApiKey') }}
          </button>
        </div>
      </div>

      <!-- 右：今日焦点 -->
      <div class="oh-focus">
        <div class="oh-focus-item">
          <span class="oh-focus-label">{{ t('dashboard.todayCost') }}</span>
          <span class="oh-focus-value oh-focus-purple">${{ formatCost(todayCost) }}</span>
        </div>
        <div class="oh-focus-item">
          <span class="oh-focus-label">{{ t('dashboard.todayRequests') }}</span>
          <span class="oh-focus-value">{{ todayRequests }}</span>
        </div>
        <div class="oh-focus-item">
          <span class="oh-focus-label">{{ t('dashboard.todayTokens') }}</span>
          <span class="oh-focus-value">{{ formatTokens(todayTokens) }}</span>
        </div>
      </div>
    </div>

    <!-- 底部微指标 -->
    <div class="relative mt-5 flex flex-wrap items-center gap-x-4 gap-y-2 border-t border-gray-200/70 pt-3 dark:border-dark-700/70">
      <span class="oh-chip">
        <Icon name="bolt" size="xs" :stroke-width="2" class="text-amber-500" />
        <b>{{ concurrency }}</b> {{ t('dashboard.concurrency') }}
      </span>
      <span class="oh-chip">
        <Icon name="trendingUp" size="xs" :stroke-width="2" class="text-sky-500" />
        <b>{{ formatTokens(rpm) }}</b> RPM
      </span>
      <span class="oh-chip">
        <Icon name="key" size="xs" :stroke-width="2" class="text-blue-500" />
        <b>{{ activeKeys }}</b> {{ t('dashboard.activeKeys') }}
      </span>
      <span class="oh-chip">
        <Icon name="clock" size="xs" :stroke-width="2" class="text-rose-500" />
        <b>{{ avgResponse }}</b> {{ t('dashboard.averageTime') }}
      </span>

      <button
        type="button"
        class="oh-refresh ml-auto"
        :disabled="loading"
        :title="t('common.refresh')"
        @click="$emit('refresh')"
      >
        <Icon name="refresh" size="xs" :stroke-width="2.2" :class="loading && 'animate-spin'" />
      </button>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import Icon from '@/components/icons/Icon.vue'

const { t } = useI18n()

interface Props {
  balance: number
  isSimple: boolean
  concurrency: number
  rpm: number
  activeKeys: number
  todayCost?: number
  todayTokens?: number
  todayRequests?: number
  avgResponseMs?: number
  loading?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  todayCost: 0,
  todayTokens: 0,
  todayRequests: 0,
  avgResponseMs: 0,
  loading: false
})

defineEmits<{
  (e: 'refresh'): void
  (e: 'navigate', path: string): void
}>()

/** 拆出整数与小数部分，整数用超大字号强调 */
const balanceParts = computed(() => {
  const v = Number(props.balance)
  const safe = Number.isFinite(v) ? v : 0
  const fixed = safe.toFixed(2)
  const [int, frac] = fixed.split('.')
  return { int: Number(int).toLocaleString(), frac }
})

const formatCost = (v: number) => {
  const n = Number(v) || 0
  return n.toFixed(n >= 1 ? 2 : 4)
}

const formatTokens = (v: number) => {
  const n = Number(v) || 0
  if (n >= 1_000_000) return `${(n / 1_000_000).toFixed(1)}M`
  if (n >= 1000) return `${(n / 1000).toFixed(1)}K`
  return String(Math.round(n))
}

const avgResponse = computed(() => {
  const ms = Number(props.avgResponseMs) || 0
  return ms >= 1000 ? `${(ms / 1000).toFixed(2)}s` : `${Math.round(ms)}ms`
})
</script>

<style scoped>
.oh {
  position: relative;
  overflow: hidden;
  border-radius: 1.25rem;
  padding: 1.5rem;
  background:
    radial-gradient(120% 120% at 0% 0%, rgba(20, 184, 166, 0.16) 0%, transparent 55%),
    linear-gradient(135deg, #ffffff 0%, #f7fcfb 100%);
  border: 1px solid rgb(20 184 166 / 0.16);
  box-shadow: 0 1px 2px rgba(15, 23, 42, 0.04), 0 24px 56px -34px rgba(13, 148, 136, 0.55);
}

:root.dark .oh {
  background:
    radial-gradient(120% 120% at 0% 0%, rgba(20, 184, 166, 0.16) 0%, transparent 55%),
    linear-gradient(135deg, #111c2e 0%, #0b1524 100%);
  border-color: rgb(20 184 166 / 0.2);
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.4), 0 24px 56px -34px rgba(0, 0, 0, 0.8);
}

.oh-glow {
  position: absolute;
  top: -7rem;
  left: -5rem;
  height: 20rem;
  width: 20rem;
  border-radius: 9999px;
  background: radial-gradient(circle, rgba(20, 184, 166, 0.22) 0%, transparent 70%);
  pointer-events: none;
}

.oh-ring {
  position: absolute;
  right: -6rem;
  bottom: -8rem;
  height: 22rem;
  width: 22rem;
  border-radius: 9999px;
  border: 1px solid rgb(20 184 166 / 0.12);
  pointer-events: none;
}

/* ============ 余额 ============ */
.oh-eyebrow {
  display: flex;
  align-items: center;
  gap: 0.375rem;
  font-size: 0.6875rem;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: rgb(13 148 136);
}

:root.dark .oh-eyebrow {
  color: rgb(94 234 212);
}

.oh-dot {
  display: inline-block;
  height: 0.375rem;
  width: 0.375rem;
  border-radius: 9999px;
  background: rgb(20 184 166);
  box-shadow: 0 0 0 3px rgb(20 184 166 / 0.18);
}

.oh-simple {
  margin-left: 0.25rem;
  border-radius: 9999px;
  padding: 0.0625rem 0.375rem;
  font-size: 0.5rem;
  letter-spacing: 0.04em;
  color: rgb(100 116 139);
  background: rgb(241 245 249);
}

.oh-amount {
  display: flex;
  align-items: baseline;
  margin-top: 0.5rem;
  font-variant-numeric: tabular-nums;
  letter-spacing: -0.045em;
}

.oh-currency {
  margin-right: 0.25rem;
  font-size: 1.5rem;
  font-weight: 600;
  color: rgb(13 148 136);
  opacity: 0.9;
}

:root.dark .oh-currency {
  color: rgb(45 212 191);
}

.oh-int {
  font-size: 3.25rem;
  line-height: 1;
  font-weight: 800;
  color: rgb(15 23 42);
}

:root.dark .oh-int {
  color: #ffffff;
}

.oh-frac {
  font-size: 1.5rem;
  font-weight: 700;
  color: rgb(148 163 184);
}

:root.dark .oh-frac {
  color: rgb(100 116 139);
}

/* ============ 按钮 ============ */
.oh-cta {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  border-radius: 0.625rem;
  padding: 0.5rem 0.9375rem;
  font-size: 0.8125rem;
  font-weight: 600;
  color: #ffffff;
  background: linear-gradient(135deg, #14b8a6 0%, #0d9488 100%);
  box-shadow: 0 6px 18px -6px rgba(13, 148, 136, 0.7);
  transition: transform 160ms ease, box-shadow 160ms ease;
}

.oh-cta:hover {
  transform: translateY(-1px);
  box-shadow: 0 10px 26px -8px rgba(13, 148, 136, 0.8);
}

.oh-cta-ghost {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  border-radius: 0.625rem;
  padding: 0.5rem 0.875rem;
  font-size: 0.8125rem;
  font-weight: 500;
  color: rgb(51 65 85);
  background: rgb(255 255 255 / 0.9);
  border: 1px solid rgb(203 213 225 / 0.9);
  transition: background-color 150ms ease;
}

.oh-cta-ghost:hover {
  background: #ffffff;
}

:root.dark .oh-cta-ghost {
  color: rgb(203 213 225);
  background: rgb(15 23 42 / 0.5);
  border-color: rgb(51 65 85);
}

/* ============ 今日焦点 ============ */
.oh-focus {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 0.5rem;
}

@media (min-width: 1024px) {
  .oh-focus {
    grid-template-columns: repeat(3, minmax(6rem, 1fr));
  }
}

.oh-focus-item {
  display: flex;
  flex-direction: column;
  gap: 0.1875rem;
  border-radius: 0.75rem;
  padding: 0.625rem 0.75rem;
  background: rgb(255 255 255 / 0.72);
  border: 1px solid rgb(226 232 240 / 0.85);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
}

:root.dark .oh-focus-item {
  background: rgb(15 23 42 / 0.5);
  border-color: rgb(51 65 85 / 0.7);
}

.oh-focus-label {
  font-size: 0.5625rem;
  font-weight: 600;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: rgb(148 163 184);
}

.oh-focus-value {
  font-size: 1.0625rem;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  color: rgb(15 23 42);
}

:root.dark .oh-focus-value {
  color: #ffffff;
}

.oh-focus-purple {
  color: rgb(147 51 234);
}

:root.dark .oh-focus-purple {
  color: rgb(192 132 252);
}

/* ============ 微指标 ============ */
.oh-chip {
  display: inline-flex;
  align-items: center;
  gap: 0.25rem;
  font-size: 0.6875rem;
  color: rgb(100 116 139);
}

:root.dark .oh-chip {
  color: rgb(148 163 184);
}

.oh-chip b {
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  color: rgb(15 23 42);
}

:root.dark .oh-chip b {
  color: #ffffff;
}

.oh-refresh {
  display: inline-flex;
  height: 1.75rem;
  width: 1.75rem;
  align-items: center;
  justify-content: center;
  border-radius: 0.5rem;
  color: rgb(100 116 139);
  background: rgb(255 255 255 / 0.9);
  border: 1px solid rgb(226 232 240);
  transition: color 150ms ease, border-color 150ms ease;
}

.oh-refresh:hover:not(:disabled) {
  color: rgb(13 148 136);
  border-color: rgb(20 184 166 / 0.4);
}

:root.dark .oh-refresh {
  color: rgb(148 163 184);
  background: rgb(15 23 42 / 0.5);
  border-color: rgb(51 65 85);
}

.oh-refresh:disabled {
  opacity: 0.55;
  cursor: not-allowed;
}
</style>
