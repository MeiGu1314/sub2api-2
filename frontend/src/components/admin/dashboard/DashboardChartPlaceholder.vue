<template>
  <!--
    图表区统一占位：加载中显示骨架，无数据显示空态（可选带操作按钮）。
    高度与真实图表保持一致，避免切换时布局跳动。
  -->
  <div :class="containerClass">
    <template v-if="loading">
      <div class="flex h-full flex-col justify-end gap-2 p-2">
        <div class="flex items-end gap-2" :style="{ height: '75%' }">
          <span
            v-for="(h, i) in bars"
            :key="i"
            class="flex-1 animate-pulse rounded-t-md bg-gray-200/80 dark:bg-dark-600/50"
            :style="{ height: `${h}%` }"
          ></span>
        </div>
        <div class="h-3 w-full animate-pulse rounded bg-gray-200/60 dark:bg-dark-600/40"></div>
      </div>
    </template>

    <div
      v-else
      class="flex h-full flex-col items-center justify-center gap-2 px-4 text-center"
    >
      <div class="dash-chart-empty-icon">
        <Icon :name="icon" size="md" :stroke-width="1.6" />
      </div>
      <p class="text-sm font-medium text-gray-600 dark:text-dark-300">
        {{ title || t('admin.dashboard.noDataAvailable') }}
      </p>
      <p v-if="description" class="max-w-xs text-xs text-gray-400 dark:text-dark-500">
        {{ description }}
      </p>
      <button
        v-if="actionText"
        type="button"
        class="mt-1 text-xs font-medium text-primary-600 hover:text-primary-500 dark:text-primary-400 dark:hover:text-primary-300"
        @click="$emit('action')"
      >
        {{ actionText }}
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import Icon from '@/components/icons/Icon.vue'

const { t } = useI18n()

/** 复用 Icon 组件声明过的图标名字面量联合 */
type IconName = InstanceType<typeof Icon>['$props']['name']

interface Props {
  loading?: boolean
  icon?: IconName
  title?: string
  description?: string
  actionText?: string
  /** 容器高度类，默认 h-64 */
  heightClass?: string
}

const props = withDefaults(defineProps<Props>(), {
  loading: false,
  icon: 'chart',
  heightClass: 'h-64'
})

defineEmits<{ (e: 'action'): void }>()

/** 固定伪随机高度，保证骨架形态稳定、不随重渲染闪动 */
const bars = [42, 68, 35, 88, 54, 72, 46, 60, 80, 38, 66, 52]

const containerClass = computed(() => [props.heightClass, 'w-full'])
</script>

<style scoped>
.dash-chart-empty-icon {
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

:root.dark .dash-chart-empty-icon {
  background: rgb(30 41 59 / 0.5);
  color: rgb(100 116 139);
  --tw-ring-color: rgb(51 65 85 / 0.6);
}
</style>
