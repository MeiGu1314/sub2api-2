<template>
  <!--
    Admin 仪表盘 KPI 卡片：在原有「图标 + 标题 + 数值 + 次要信息」结构上做视觉增强。
    纯展示组件，不发起请求，所有数值由父组件计算好后传入。
  -->
  <div class="kpi-card group">
    <!-- 装饰性光晕，不参与交互 -->
    <span class="kpi-orb" aria-hidden="true"></span>

    <div class="relative flex items-start gap-3">
      <!-- 图标徽章 -->
      <div class="kpi-icon">
        <Icon :name="icon" size="md" :stroke-width="2" />
      </div>

      <!-- 主体 -->
      <div class="min-w-0 flex-1">
        <p class="kpi-label truncate">{{ label }}</p>

        <div
          v-if="loading"
          class="mt-2 h-7 w-24 animate-pulse rounded-md bg-gray-200/80 dark:bg-dark-600/60"
        ></div>
        <p v-else class="kpi-value" :title="valueTitle || String(value)">
          {{ value }}
        </p>

        <!-- 次要信息行 -->
        <div class="mt-1.5 flex flex-wrap items-center gap-x-2 gap-y-1">
          <template v-if="loading">
            <span class="h-3 w-16 animate-pulse rounded bg-gray-200/70 dark:bg-dark-600/50"></span>
          </template>
          <template v-else>
            <slot name="meta">
              <span v-if="meta" class="kpi-meta">{{ meta }}</span>
            </slot>
          </template>
        </div>
      </div>

      <!-- 右上角强调数值（如 RPM/TPM 的次行） -->
      <div v-if="accent !== undefined && !loading" class="shrink-0 text-right">
        <p class="text-sm font-semibold tabular-nums" :class="accentClass">{{ accent }}</p>
        <p v-if="accentLabel" class="text-[10px] uppercase tracking-wide text-gray-400 dark:text-dark-500">
          {{ accentLabel }}
        </p>
      </div>
    </div>

    <!-- 底部细进度条（可选，用于表示活跃占比等） -->
    <div
      v-if="progress !== undefined && !loading"
      class="relative mt-3 h-1 overflow-hidden rounded-full bg-gray-100 dark:bg-dark-700/70"
      role="presentation"
    >
      <span
        class="block h-full rounded-full transition-[width] duration-700 ease-out"
        :class="progressBarClass"
        :style="{ width: `${clampedProgress}%` }"
      ></span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import Icon from '@/components/icons/Icon.vue'

/** 复用 Icon 组件声明过的图标名字面量联合，避免引入未定义的图标名 */
type IconName = InstanceType<typeof Icon>['$props']['name']

export type KpiTone =
  | 'blue'
  | 'purple'
  | 'green'
  | 'emerald'
  | 'amber'
  | 'indigo'
  | 'violet'
  | 'rose'

interface Props {
  icon: IconName
  label: string
  value: string | number
  meta?: string
  /** 右上角强调值（如 TPM） */
  accent?: string | number
  accentLabel?: string
  /** 0-100，传入后显示底部进度条 */
  progress?: number
  tone?: KpiTone
  loading?: boolean
  valueTitle?: string
}

const props = withDefaults(defineProps<Props>(), {
  tone: 'blue',
  loading: false
})

/** Tailwind 需要静态类名才能在构建时被扫描到，因此显式映射而非拼接。 */
const TONE_CLASSES: Record<KpiTone, { icon: string; bar: string; accent: string }> = {
  blue: {
    icon: 'bg-blue-500/10 text-blue-600 ring-blue-500/20 dark:bg-blue-500/15 dark:text-blue-400',
    bar: 'bg-gradient-to-r from-blue-400 to-blue-600',
    accent: 'text-blue-600 dark:text-blue-400'
  },
  purple: {
    icon: 'bg-purple-500/10 text-purple-600 ring-purple-500/20 dark:bg-purple-500/15 dark:text-purple-400',
    bar: 'bg-gradient-to-r from-purple-400 to-purple-600',
    accent: 'text-purple-600 dark:text-purple-400'
  },
  green: {
    icon: 'bg-green-500/10 text-green-600 ring-green-500/20 dark:bg-green-500/15 dark:text-green-400',
    bar: 'bg-gradient-to-r from-green-400 to-green-600',
    accent: 'text-green-600 dark:text-green-400'
  },
  emerald: {
    icon: 'bg-emerald-500/10 text-emerald-600 ring-emerald-500/20 dark:bg-emerald-500/15 dark:text-emerald-400',
    bar: 'bg-gradient-to-r from-emerald-400 to-emerald-600',
    accent: 'text-emerald-600 dark:text-emerald-400'
  },
  amber: {
    icon: 'bg-amber-500/10 text-amber-600 ring-amber-500/20 dark:bg-amber-500/15 dark:text-amber-400',
    bar: 'bg-gradient-to-r from-amber-400 to-amber-600',
    accent: 'text-amber-600 dark:text-amber-400'
  },
  indigo: {
    icon: 'bg-indigo-500/10 text-indigo-600 ring-indigo-500/20 dark:bg-indigo-500/15 dark:text-indigo-400',
    bar: 'bg-gradient-to-r from-indigo-400 to-indigo-600',
    accent: 'text-indigo-600 dark:text-indigo-400'
  },
  violet: {
    icon: 'bg-violet-500/10 text-violet-600 ring-violet-500/20 dark:bg-violet-500/15 dark:text-violet-400',
    bar: 'bg-gradient-to-r from-violet-400 to-violet-600',
    accent: 'text-violet-600 dark:text-violet-400'
  },
  rose: {
    icon: 'bg-rose-500/10 text-rose-600 ring-rose-500/20 dark:bg-rose-500/15 dark:text-rose-400',
    bar: 'bg-gradient-to-r from-rose-400 to-rose-600',
    accent: 'text-rose-600 dark:text-rose-400'
  }
}

const accentClass = computed(() => TONE_CLASSES[props.tone].accent)
const progressBarClass = computed(() => TONE_CLASSES[props.tone].bar)

const clampedProgress = computed(() => {
  const raw = Number(props.progress)
  if (!Number.isFinite(raw)) return 0
  return Math.min(100, Math.max(0, raw))
})
</script>

<style scoped>
.kpi-card {
  position: relative;
  overflow: hidden;
  padding: 1rem;
  border-radius: 1rem;
  background: rgb(255 255 255 / 0.72);
  backdrop-filter: blur(14px) saturate(1.15);
  -webkit-backdrop-filter: blur(14px) saturate(1.15);
  border: 1px solid rgb(255 255 255 / 0.65);
  box-shadow: 0 1px 2px rgba(15, 23, 42, 0.04), 0 8px 24px -12px rgba(15, 23, 42, 0.12);
  transition: transform 300ms ease, box-shadow 300ms ease, border-color 300ms ease;
}

.kpi-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 2px 4px rgba(15, 23, 42, 0.05), 0 18px 40px -16px rgba(15, 23, 42, 0.2);
}

:root.dark .kpi-card {
  background: rgb(30 41 59 / 0.55);
  border-color: rgb(148 163 184 / 0.12);
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.3), 0 8px 24px -12px rgba(0, 0, 0, 0.5);
}

:root.dark .kpi-card:hover {
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.35), 0 18px 40px -16px rgba(0, 0, 0, 0.6);
}

/* 右下角柔光，让卡片在暗色下也有层次 */
.kpi-orb {
  position: absolute;
  right: -2.5rem;
  bottom: -3rem;
  height: 7rem;
  width: 7rem;
  border-radius: 9999px;
  background: radial-gradient(circle, rgb(20 184 166 / 0.18) 0%, transparent 70%);
  opacity: 0;
  transition: opacity 300ms ease;
  pointer-events: none;
}

.kpi-card:hover .kpi-orb {
  opacity: 1;
}

.kpi-icon {
  display: flex;
  height: 2.5rem;
  width: 2.5rem;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  border-radius: 0.75rem;
  @apply ring-1;
}

.kpi-label {
  font-size: 0.75rem;
  font-weight: 500;
  color: rgb(107 114 128);
}

:root.dark .kpi-label {
  color: rgb(148 163 184);
}

.kpi-value {
  margin-top: 0.125rem;
  font-size: 1.5rem;
  line-height: 2rem;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  letter-spacing: -0.02em;
  color: rgb(17 24 39);
}

:root.dark .kpi-value {
  color: rgb(255 255 255);
}

.kpi-meta {
  font-size: 0.6875rem;
  color: rgb(107 114 128);
}

:root.dark .kpi-meta {
  color: rgb(148 163 184);
}
</style>
