<template>
  <div class="app-shell relative min-h-screen">
    <!-- 页面背景：渐变底 + 网格 + 漂浮光球，纯装饰不参与交互 -->
    <div class="app-bg" aria-hidden="true">
      <span class="app-bg-orb app-bg-orb-teal"></span>
      <span class="app-bg-orb app-bg-orb-blue"></span>
      <span class="app-bg-orb app-bg-orb-violet"></span>
      <span class="app-bg-grid"></span>
    </div>

    <!-- Sidebar -->
    <AppSidebar />

    <!-- Main Content Area -->
    <div
      class="relative min-h-screen transition-all duration-300"
      :class="[sidebarCollapsed ? 'lg:ml-[72px]' : 'lg:ml-64']"
    >
      <!-- Header -->
      <AppHeader />

      <!-- Main Content -->
      <main class="relative z-10 p-4 md:p-6 lg:p-8">
        <slot />
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import '@/styles/onboarding.css'
import { computed, onMounted } from 'vue'
import { useAppStore } from '@/stores'
import { useAuthStore } from '@/stores/auth'
import { useOnboardingTour } from '@/composables/useOnboardingTour'
import { useOnboardingStore } from '@/stores/onboarding'
import AppSidebar from './AppSidebar.vue'
import AppHeader from './AppHeader.vue'

const appStore = useAppStore()
const authStore = useAuthStore()
const sidebarCollapsed = computed(() => appStore.sidebarCollapsed)
const isAdmin = computed(() => authStore.user?.role === 'admin')

const { replayTour } = useOnboardingTour({
  storageKey: isAdmin.value ? 'admin_guide' : 'user_guide',
  autoStart: true
})

const onboardingStore = useOnboardingStore()

onMounted(() => {
  onboardingStore.setReplayCallback(replayTour)
})

defineExpose({ replayTour })
</script>

<style scoped>
/* ============ 页面背景 ============ */
/* 主内容区：接近白的浅色，与浅灰蓝侧边栏形成轻微层次色差 */
.app-shell {
  background-image: linear-gradient(180deg, #fbfcfd 0%, #f7fafb 55%, #f4f8f9 100%);
}

:root.dark .app-shell {
  background-image: linear-gradient(180deg, #060d1a 0%, #0a1524 55%, #0d1b2b 100%);
}

.app-bg {
  position: fixed;
  inset: 0;
  z-index: 0;
  overflow: hidden;
  pointer-events: none;
}

/* 网格：顶部渐隐，避免整体过密 */
.app-bg-grid {
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(to right, rgba(15, 23, 42, 0.045) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(15, 23, 42, 0.045) 1px, transparent 1px);
  background-size: 48px 48px;
  -webkit-mask-image: radial-gradient(ellipse 90% 65% at 50% 0%, #000 30%, transparent 100%);
  mask-image: radial-gradient(ellipse 90% 65% at 50% 0%, #000 30%, transparent 100%);
}

:root.dark .app-bg-grid {
  background-image:
    linear-gradient(to right, rgba(148, 163, 184, 0.07) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(148, 163, 184, 0.07) 1px, transparent 1px);
}

/* 光球：柔和色彩点缀 */
.app-bg-orb {
  position: absolute;
  border-radius: 9999px;
  filter: blur(90px);
  opacity: 0.6;
  animation: appBgFloat 20s ease-in-out infinite;
  will-change: transform;
}

.app-bg-orb-teal {
  width: 34rem;
  height: 34rem;
  top: -13rem;
  left: -9rem;
  background: radial-gradient(circle, rgba(20, 184, 166, 0.38) 0%, transparent 70%);
}

.app-bg-orb-blue {
  width: 30rem;
  height: 30rem;
  top: -7rem;
  right: -11rem;
  background: radial-gradient(circle, rgba(59, 130, 246, 0.3) 0%, transparent 70%);
  animation-delay: -7s;
}

.app-bg-orb-violet {
  width: 26rem;
  height: 26rem;
  bottom: -13rem;
  left: 34%;
  background: radial-gradient(circle, rgba(168, 85, 247, 0.22) 0%, transparent 70%);
  animation-delay: -14s;
}

:root.dark .app-bg-orb {
  opacity: 0.4;
}

@keyframes appBgFloat {
  0%,
  100% {
    transform: translate3d(0, 0, 0);
  }
  50% {
    transform: translate3d(0, -26px, 0);
  }
}

@media (prefers-reduced-motion: reduce) {
  .app-bg-orb {
    animation: none;
  }
}
</style>
