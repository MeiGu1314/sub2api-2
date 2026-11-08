<template>
  <!--
    轻量指标芯片：用于页面顶部的次级统计条（比 KPI 卡更紧凑）。
    结构：小图标 + 标签 + 数值，可选后缀说明。
  -->
  <div class="stat-chip" :class="toneClass">
    <span v-if="icon" class="stat-chip-icon">
      <Icon :name="icon" size="sm" :stroke-width="2" />
    </span>

    <span class="min-w-0 flex-1">
      <span class="stat-chip-label">{{ label }}</span>
      <span class="stat-chip-value">
        {{ value }}<span v-if="suffix" class="stat-chip-suffix">{{ suffix }}</span>
      </span>
    </span>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import Icon from '@/components/icons/Icon.vue'

/** 复用 Icon 组件声明过的图标名字面量联合 */
type IconName = InstanceType<typeof Icon>['$props']['name']

type ChipTone = 'teal' | 'blue' | 'purple' | 'amber' | 'emerald' | 'rose' | 'slate'

interface Props {
  label: string
  value: string | number
  suffix?: string
  icon?: IconName
  tone?: ChipTone
}

const props = withDefaults(defineProps<Props>(), {
  tone: 'teal'
})

const TONE_CLASSES: Record<ChipTone, { wrap: string; icon: string; value: string }> = {
  teal: {
    wrap: 'ring-primary-500/12',
    icon: 'bg-primary-500/10 text-primary-600 dark:bg-primary-500/15 dark:text-primary-400',
    value: 'text-gray-900 dark:text-white'
  },
  blue: {
    wrap: 'ring-blue-500/12',
    icon: 'bg-blue-500/10 text-blue-600 dark:bg-blue-500/15 dark:text-blue-400',
    value: 'text-gray-900 dark:text-white'
  },
  purple: {
    wrap: 'ring-purple-500/12',
    icon: 'bg-purple-500/10 text-purple-600 dark:bg-purple-500/15 dark:text-purple-400',
    value: 'text-gray-900 dark:text-white'
  },
  amber: {
    wrap: 'ring-amber-500/12',
    icon: 'bg-amber-500/10 text-amber-600 dark:bg-amber-500/15 dark:text-amber-400',
    value: 'text-gray-900 dark:text-white'
  },
  emerald: {
    wrap: 'ring-emerald-500/12',
    icon: 'bg-emerald-500/10 text-emerald-600 dark:bg-emerald-500/15 dark:text-emerald-400',
    value: 'text-emerald-700 dark:text-emerald-300'
  },
  rose: {
    wrap: 'ring-rose-500/12',
    icon: 'bg-rose-500/10 text-rose-600 dark:bg-rose-500/15 dark:text-rose-400',
    value: 'text-rose-700 dark:text-rose-300'
  },
  slate: {
    wrap: 'ring-gray-400/12',
    icon: 'bg-gray-400/10 text-gray-600 dark:bg-gray-500/15 dark:text-gray-400',
    value: 'text-gray-900 dark:text-white'
  }
}

const toneClass = computed(() => TONE_CLASSES[props.tone])
</script>

<style scoped>
.stat-chip {
  display: flex;
  min-width: 0;
  align-items: center;
  gap: 0.625rem;
  border-radius: 0.75rem;
  padding: 0.625rem 0.875rem;
  background: rgb(255 255 255);
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
  @apply ring-1;
}

:root.dark .stat-chip {
  background: rgb(30 41 59 / 0.5);
}

.stat-chip-icon {
  display: inline-flex;
  height: 1.75rem;
  width: 1.75rem;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  border-radius: 0.5rem;
}

.stat-chip-label {
  display: block;
  font-size: 0.625rem;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  color: rgb(107 114 128);
}

:root.dark .stat-chip-label {
  color: rgb(148 163 184);
}

.stat-chip-value {
  display: block;
  font-size: 1rem;
  font-weight: 700;
  line-height: 1.375rem;
  font-variant-numeric: tabular-nums;
  letter-spacing: -0.01em;
}

.stat-chip-suffix {
  margin-left: 0.1875rem;
  font-size: 0.6875rem;
  font-weight: 500;
  color: rgb(107 114 128);
}

:root.dark .stat-chip-suffix {
  color: rgb(148 163 184);
}
</style>
