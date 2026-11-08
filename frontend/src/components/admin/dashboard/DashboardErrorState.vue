<template>
  <!--
    仪表盘错误态：区分「首次加载失败」（整页接管）和「局部刷新失败」（紧凑横幅）。
    父组件据此决定是否展示已有数据。
  -->
  <div
    v-if="variant === 'page'"
    class="dash-error-page"
    role="alert"
  >
    <div class="dash-error-glow" aria-hidden="true"></div>

    <div class="dash-error-icon">
      <Icon name="exclamationTriangle" size="lg" :stroke-width="1.8" />
    </div>

    <h3 class="dash-error-title">{{ title || t('admin.dashboard.errorState.title') }}</h3>

    <p class="dash-error-desc">
      <slot name="description">
        {{ message || t('admin.dashboard.errorState.description') }}
      </slot>
    </p>

    <div class="mt-5 flex flex-wrap items-center justify-center gap-2">
      <button
        type="button"
        class="btn btn-primary inline-flex items-center gap-1.5"
        :disabled="retrying"
        @click="$emit('retry')"
      >
        <Icon
          name="refresh"
          size="sm"
          :class="retrying && 'animate-spin'"
        />
        {{ retrying ? t('admin.dashboard.errorState.retrying') : t('admin.dashboard.errorState.retry') }}
      </button>
      <slot name="actions"></slot>
    </div>
  </div>

  <div v-else class="dash-error-banner" role="alert">
    <Icon name="exclamationCircle" size="md" class="mt-0.5 flex-shrink-0 text-red-500" />
    <div class="min-w-0 flex-1">
      <p class="text-sm font-medium text-red-700 dark:text-red-300">
        {{ title || t('admin.dashboard.errorState.partialTitle') }}
      </p>
      <p v-if="message" class="mt-0.5 text-xs text-red-600/80 dark:text-red-300/70">
        {{ message }}
      </p>
    </div>
    <button
      type="button"
      class="dash-error-retry"
      :disabled="retrying"
      @click="$emit('retry')"
    >
      <Icon name="refresh" size="sm" :class="retrying && 'animate-spin'" />
      <span>{{ t('admin.dashboard.errorState.retry') }}</span>
    </button>
  </div>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n'
import Icon from '@/components/icons/Icon.vue'

const { t } = useI18n()

interface Props {
  variant?: 'page' | 'inline'
  title?: string
  message?: string
  retrying?: boolean
}

withDefaults(defineProps<Props>(), {
  variant: 'inline',
  retrying: false
})

defineEmits<{ (e: 'retry'): void }>()
</script>

<style scoped>
.dash-error-page {
  position: relative;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  align-items: center;
  border-radius: 1rem;
  padding: 3.5rem 1.5rem;
  text-align: center;
  background: rgb(255 255 255);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.04), 0 1px 2px rgba(0, 0, 0, 0.06);
  @apply ring-1 ring-red-100;
}

:root.dark .dash-error-page {
  background: rgb(30 41 59 / 0.5);
  --tw-ring-color: rgb(127 29 29 / 0.3);
}

.dash-error-glow {
  position: absolute;
  top: -6rem;
  left: 50%;
  height: 14rem;
  width: 28rem;
  transform: translateX(-50%);
  border-radius: 9999px;
  background: radial-gradient(circle, rgb(239 68 68 / 0.14) 0%, transparent 70%);
  pointer-events: none;
}

.dash-error-icon {
  position: relative;
  display: flex;
  height: 4rem;
  width: 4rem;
  align-items: center;
  justify-content: center;
  border-radius: 1rem;
  color: rgb(220 38 38);
  background: rgb(254 242 242);
  @apply ring-1 ring-red-200;
}

:root.dark .dash-error-icon {
  background: rgb(127 29 29 / 0.25);
  color: rgb(248 113 113);
  --tw-ring-color: rgb(153 27 27 / 0.4);
}

.dash-error-title {
  position: relative;
  margin-top: 1.25rem;
  font-size: 1rem;
  font-weight: 600;
  color: rgb(17 24 39);
}

:root.dark .dash-error-title {
  color: rgb(255 255 255);
}

.dash-error-desc {
  position: relative;
  margin-top: 0.5rem;
  max-width: 32rem;
  font-size: 0.875rem;
  line-height: 1.5rem;
  color: rgb(107 114 128);
}

:root.dark .dash-error-desc {
  color: rgb(148 163 184);
}

.dash-error-banner {
  display: flex;
  align-items: flex-start;
  gap: 0.75rem;
  border-radius: 0.75rem;
  padding: 0.875rem 1rem;
  background: rgb(254 242 242);
  @apply ring-1 ring-red-200;
}

:root.dark .dash-error-banner {
  background: rgb(127 29 29 / 0.2);
  --tw-ring-color: rgb(153 27 27 / 0.4);
}

.dash-error-retry {
  display: inline-flex;
  flex-shrink: 0;
  align-items: center;
  gap: 0.375rem;
  border-radius: 0.5rem;
  padding: 0.25rem 0.625rem;
  font-size: 0.75rem;
  font-weight: 500;
  color: rgb(185 28 28);
  background: rgb(255 255 255 / 0.7);
  transition: background-color 150ms ease;
}

.dash-error-retry:hover:not(:disabled) {
  background: rgb(255 255 255);
}

:root.dark .dash-error-retry {
  color: rgb(252 165 165);
  background: rgb(30 41 59 / 0.6);
}

:root.dark .dash-error-retry:hover:not(:disabled) {
  background: rgb(30 41 59);
}

.dash-error-retry:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
</style>
