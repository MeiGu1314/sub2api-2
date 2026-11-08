<template>
  <!--
    新用户引导卡：当用户还没有任何用量时替代空图表区。
    提供清晰的下一步动作，而不是展示一堆 0。
  -->
  <section class="gt">
    <span class="gt-orb" aria-hidden="true"></span>

    <div class="relative flex flex-col gap-5 lg:flex-row lg:items-center">
      <!-- 左侧文案 -->
      <div class="min-w-0 flex-1">
        <span class="gt-badge">
          <Icon name="sparkles" size="xs" :stroke-width="2.5" />
          {{ t('dashboard.getStarted.badge') }}
        </span>

        <h2 class="gt-title">{{ t('dashboard.getStarted.title') }}</h2>
        <p class="gt-desc">{{ t('dashboard.getStarted.description') }}</p>

        <div class="mt-4 flex flex-wrap items-center gap-2">
          <button type="button" class="gt-btn-primary" @click="$emit('navigate', '/keys')">
            <Icon name="key" size="sm" :stroke-width="2.5" />
            {{ t('dashboard.getStarted.createKey') }}
          </button>
          <button type="button" class="gt-btn-ghost" @click="$emit('navigate', '/redeem')">
            <Icon name="gift" size="sm" :stroke-width="2" />
            {{ t('dashboard.redeemCode') }}
          </button>
        </div>
      </div>

      <!-- 右侧三步 -->
      <ol class="gt-steps">
        <li class="gt-step">
          <span class="gt-step-num">1</span>
          <span class="min-w-0">
            <span class="gt-step-title">{{ t('dashboard.getStarted.step1Title') }}</span>
            <span class="gt-step-desc">{{ t('dashboard.getStarted.step1Desc') }}</span>
          </span>
        </li>
        <li class="gt-step">
          <span class="gt-step-num">2</span>
          <span class="min-w-0">
            <span class="gt-step-title">{{ t('dashboard.getStarted.step2Title') }}</span>
            <span class="gt-step-desc">{{ t('dashboard.getStarted.step2Desc') }}</span>
          </span>
        </li>
        <li class="gt-step">
          <span class="gt-step-num">3</span>
          <span class="min-w-0">
            <span class="gt-step-title">{{ t('dashboard.getStarted.step3Title') }}</span>
            <span class="gt-step-desc">{{ t('dashboard.getStarted.step3Desc') }}</span>
          </span>
        </li>
      </ol>
    </div>
  </section>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n'
import Icon from '@/components/icons/Icon.vue'

defineEmits<{ (e: 'navigate', path: string): void }>()
const { t } = useI18n()
</script>

<style scoped>
.gt {
  position: relative;
  overflow: hidden;
  border-radius: 1rem;
  padding: 1.25rem 1.375rem;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.94) 0%, rgba(240, 253, 250, 0.9) 100%);
  border: 1px solid rgb(255 255 255 / 0.75);
  box-shadow: 0 1px 2px rgba(15, 23, 42, 0.04), 0 14px 36px -26px rgba(13, 148, 136, 0.5);
  backdrop-filter: blur(16px) saturate(1.2);
  -webkit-backdrop-filter: blur(16px) saturate(1.2);
}

:root.dark .gt {
  background: linear-gradient(135deg, rgba(30, 41, 59, 0.8) 0%, rgba(19, 78, 74, 0.36) 100%);
  border-color: rgb(148 163 184 / 0.14);
}

.gt-orb {
  position: absolute;
  top: -5rem;
  left: -3rem;
  height: 13rem;
  width: 13rem;
  border-radius: 9999px;
  background: radial-gradient(circle, rgba(20, 184, 166, 0.2) 0%, transparent 70%);
  pointer-events: none;
}

.gt-badge {
  display: inline-flex;
  align-items: center;
  gap: 0.3125rem;
  border-radius: 9999px;
  padding: 0.1875rem 0.5625rem;
  font-size: 0.625rem;
  font-weight: 700;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  color: rgb(15 118 110);
  background: rgb(255 255 255 / 0.9);
  border: 1px solid rgb(20 184 166 / 0.25);
}

:root.dark .gt-badge {
  color: rgb(94 234 212);
  background: rgb(19 78 74 / 0.6);
  border-color: rgb(20 184 166 / 0.3);
}

.gt-title {
  margin-top: 0.625rem;
  font-size: 1.125rem;
  line-height: 1.6rem;
  font-weight: 700;
  letter-spacing: -0.01em;
  color: rgb(15 23 42);
}

:root.dark .gt-title {
  color: rgb(255 255 255);
}

.gt-desc {
  margin-top: 0.25rem;
  max-width: 34rem;
  font-size: 0.8125rem;
  line-height: 1.375rem;
  color: rgb(100 116 139);
}

:root.dark .gt-desc {
  color: rgb(148 163 184);
}

.gt-btn-primary {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  border-radius: 0.625rem;
  padding: 0.5rem 0.9375rem;
  font-size: 0.8125rem;
  font-weight: 600;
  color: rgb(255 255 255);
  background: linear-gradient(135deg, #14b8a6 0%, #0d9488 100%);
  box-shadow: 0 4px 14px -4px rgba(13, 148, 136, 0.6);
  transition: transform 160ms ease, box-shadow 160ms ease;
}

.gt-btn-primary:hover {
  transform: translateY(-1px);
  box-shadow: 0 8px 22px -6px rgba(13, 148, 136, 0.7);
}

.gt-btn-ghost {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  border-radius: 0.625rem;
  padding: 0.5rem 0.875rem;
  font-size: 0.8125rem;
  font-weight: 500;
  color: rgb(71 85 105);
  background: rgb(255 255 255 / 0.85);
  border: 1px solid rgb(226 232 240);
  transition: background-color 150ms ease;
}

.gt-btn-ghost:hover {
  background: #ffffff;
}

:root.dark .gt-btn-ghost {
  color: rgb(203 213 225);
  background: rgb(15 23 42 / 0.5);
  border-color: rgb(51 65 85);
}

.gt-steps {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  flex-shrink: 0;
  width: 100%;
  max-width: 22rem;
}

.gt-step {
  display: flex;
  align-items: flex-start;
  gap: 0.625rem;
  border-radius: 0.625rem;
  padding: 0.5625rem 0.6875rem;
  background: rgb(255 255 255 / 0.72);
  border: 1px solid rgb(226 232 240 / 0.85);
}

:root.dark .gt-step {
  background: rgb(15 23 42 / 0.42);
  border-color: rgb(51 65 85 / 0.7);
}

.gt-step-num {
  display: inline-flex;
  height: 1.375rem;
  width: 1.375rem;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  border-radius: 9999px;
  font-size: 0.6875rem;
  font-weight: 700;
  color: rgb(255 255 255);
  background: linear-gradient(135deg, #14b8a6 0%, #0d9488 100%);
}

.gt-step-title {
  display: block;
  font-size: 0.75rem;
  font-weight: 600;
  color: rgb(15 23 42);
}

:root.dark .gt-step-title {
  color: rgb(255 255 255);
}

.gt-step-desc {
  display: block;
  margin-top: 0.0625rem;
  font-size: 0.6875rem;
  line-height: 1.0625rem;
  color: rgb(148 163 184);
}

:root.dark .gt-step-desc {
  color: rgb(100 116 139);
}
</style>
