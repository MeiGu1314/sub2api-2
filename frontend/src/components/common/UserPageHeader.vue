<template>
  <!--
    用户端页面统一头部：渐变底纹 + 图标徽章 + 标题/描述 + 右侧操作槽。
    与管理仪表盘的 dash-topbar 保持同一视觉语言。
  -->
  <header class="user-page-header" :class="{ 'user-page-header-compact': compact }">
    <!-- 装饰性光晕 -->
    <span class="user-page-header-glow" aria-hidden="true"></span>

    <div class="relative flex min-w-0 flex-1 items-start gap-3">
      <span v-if="icon" class="user-page-header-icon">
        <Icon :name="icon" size="md" :stroke-width="2" />
      </span>

      <div class="min-w-0">
        <div class="flex flex-wrap items-center gap-2">
          <h1 class="truncate text-lg font-semibold text-gray-900 dark:text-white">
            {{ title }}
          </h1>
          <slot name="badges"></slot>
        </div>

        <p v-if="description || $slots.description" class="mt-0.5 flex flex-wrap items-center gap-x-2 gap-y-1 text-xs text-gray-500 dark:text-dark-400">
          <slot name="description">{{ description }}</slot>
        </p>
      </div>
    </div>

    <div v-if="$slots.actions" class="relative flex flex-shrink-0 flex-wrap items-center gap-2">
      <slot name="actions"></slot>
    </div>
  </header>
</template>

<script setup lang="ts">
import Icon from '@/components/icons/Icon.vue'

/** 复用 Icon 组件声明过的图标名字面量联合 */
type IconName = InstanceType<typeof Icon>['$props']['name']

interface Props {
  title: string
  description?: string
  icon?: IconName
  /** 紧凑模式：减少上下内边距 */
  compact?: boolean
}

withDefaults(defineProps<Props>(), {
  compact: false
})
</script>

<style scoped>
.user-page-header {
  position: relative;
  overflow: hidden;
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: 0.75rem;
  border-radius: 1rem;
  padding: 1rem 1.125rem;
  background: linear-gradient(135deg, rgba(20, 184, 166, 0.09) 0%, rgba(6, 182, 212, 0.04) 100%);
  @apply ring-1 ring-primary-500/10;
}

.user-page-header-compact {
  padding: 0.75rem 1rem;
}

:root.dark .user-page-header {
  background: linear-gradient(135deg, rgba(20, 184, 166, 0.11) 0%, rgba(6, 182, 212, 0.04) 100%);
  --tw-ring-color: rgb(20 184 166 / 0.15);
}

.user-page-header-glow {
  position: absolute;
  right: -3rem;
  top: -4rem;
  height: 10rem;
  width: 16rem;
  border-radius: 9999px;
  background: radial-gradient(circle, rgb(20 184 166 / 0.16) 0%, transparent 70%);
  pointer-events: none;
}

.user-page-header-icon {
  display: inline-flex;
  height: 2.25rem;
  width: 2.25rem;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  border-radius: 0.75rem;
  color: rgb(13 148 136);
  background: rgb(255 255 255 / 0.85);
  box-shadow: 0 1px 2px rgba(13, 148, 136, 0.12);
  @apply ring-1 ring-primary-500/20;
}

:root.dark .user-page-header-icon {
  color: rgb(94 234 212);
  background: rgb(19 78 74 / 0.6);
  --tw-ring-color: rgb(20 184 166 / 0.3);
}
</style>
