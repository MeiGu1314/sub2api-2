<template>
  <!-- 自定义首页内容（保持原有三种模式） -->
  <div v-if="hasHomeContent" class="min-h-screen">
    <iframe
      v-if="isHomeContentUrl"
      :src="homeContent.trim()"
      class="h-screen w-full border-0"
      allowfullscreen
    ></iframe>
    <!-- SECURITY: homeContent 为管理员配置项，这里按设计接受其 XSS 风险 -->
    <div v-else v-html="homeContent"></div>
  </div>

  <!-- 紧凑首页 -->
  <div
    v-else-if="compactHomeEnabled"
    data-testid="compact-home"
    class="flex min-h-screen flex-col bg-gray-50 text-gray-900 dark:bg-dark-950 dark:text-white"
  >
    <header class="border-b border-gray-200 px-4 py-4 sm:px-6 dark:border-dark-800">
      <nav class="mx-auto flex max-w-5xl flex-wrap items-center justify-between gap-3 sm:gap-4">
        <div class="flex min-w-0 flex-1 items-center gap-3">
          <img :src="siteLogo || '/logo.svg'" alt="Logo" class="h-9 w-9 shrink-0 rounded-lg object-contain" />
          <span class="min-w-0 truncate text-base font-semibold">{{ siteName }}</span>
        </div>
        <div class="flex max-w-full shrink-0 flex-wrap items-center justify-end gap-2">
          <LocaleSwitcher />
          <a
            v-if="docUrl"
            :href="docUrl"
            target="_blank"
            rel="noopener noreferrer"
            class="flex h-10 w-10 shrink-0 items-center justify-center rounded-lg text-gray-500 hover:bg-gray-100 dark:text-dark-400 dark:hover:bg-dark-800"
            :title="t('home.viewDocs')"
          >
            <Icon name="book" size="md" />
          </a>
          <router-link
            v-if="showModelPlazaEntry"
            to="/model-plaza"
            class="flex h-10 shrink-0 items-center gap-1.5 rounded-lg px-2.5 text-sm font-medium text-gray-500 hover:bg-gray-100 hover:text-gray-700 dark:text-dark-400 dark:hover:bg-dark-800 dark:hover:text-white"
            :title="t('nav.modelPlaza')"
          >
            <Icon name="grid" size="md" />
            <span class="hidden sm:inline">{{ t('nav.modelPlaza') }}</span>
          </router-link>
          <button
            class="flex h-10 w-10 shrink-0 items-center justify-center rounded-lg text-gray-500 hover:bg-gray-100 dark:text-dark-400 dark:hover:bg-dark-800"
            :title="isDark ? t('home.switchToLight') : t('home.switchToDark')"
            @click="toggleTheme"
          >
            <Icon v-if="isDark" name="sun" size="md" />
            <Icon v-else name="moon" size="md" />
          </button>
          <router-link
            :to="isAuthenticated ? dashboardPath : '/login'"
            class="inline-flex min-h-10 shrink-0 items-center justify-center rounded-lg bg-gray-900 px-4 py-2 text-sm font-medium text-white hover:bg-gray-800 dark:bg-white dark:text-gray-900 dark:hover:bg-gray-200"
          >
            {{ isAuthenticated ? t('home.dashboard') : t('home.login') }}
          </router-link>
        </div>
      </nav>
    </header>

    <main class="flex min-w-0 flex-1 items-center justify-center px-4 py-16 sm:px-6">
      <div class="min-w-0 max-w-2xl text-center">
        <img :src="siteLogo || '/logo.svg'" alt="Logo" class="mx-auto mb-6 h-20 w-20 rounded-2xl object-contain" />
        <h1 class="[overflow-wrap:anywhere] text-3xl font-bold md:text-4xl">{{ siteName }}</h1>
        <p class="mt-4 whitespace-pre-wrap [overflow-wrap:anywhere] text-base text-gray-600 dark:text-dark-300">
          {{ siteSubtitle }}
        </p>
        <router-link
          :to="isAuthenticated ? dashboardPath : '/login'"
          class="mt-8 inline-flex min-h-10 items-center justify-center rounded-lg bg-primary-600 px-5 py-2.5 text-sm font-medium text-white hover:bg-primary-700"
        >
          {{ isAuthenticated ? t('home.goToDashboard') : t('home.login') }}
        </router-link>
      </div>
    </main>

    <footer class="min-w-0 border-t border-gray-200 px-4 py-5 text-center text-sm text-gray-500 [overflow-wrap:anywhere] sm:px-6 dark:border-dark-800 dark:text-dark-400">
      &copy; {{ currentYear }} {{ siteName }}
    </footer>
  </div>

  <!-- ============ 默认首页（深色科技风） ============ -->
  <div v-else class="dk-root">
    <!-- 背景 -->
    <div class="dk-bg" aria-hidden="true">
      <span class="dk-beam dk-beam-a"></span>
      <span class="dk-beam dk-beam-b"></span>
      <span class="dk-glow"></span>
      <span class="dk-grid"></span>
      <span class="dk-noise"></span>
    </div>

    <!-- 顶栏 -->
    <header class="dk-header">
      <nav class="dk-nav">
        <router-link to="/home" class="dk-brand">
          <span class="dk-brand-logo">
            <img :src="siteLogo || '/logo.svg'" alt="Logo" class="h-full w-full object-contain" />
          </span>
          <span class="dk-brand-name">{{ siteName }}</span>
        </router-link>

        <div class="dk-nav-links">
          <a href="#features" class="dk-nav-link">{{ t('home.navFeatures') }}</a>
          <a href="#providers" class="dk-nav-link">{{ t('home.navProviders') }}</a>
          <a href="#stats" class="dk-nav-link">{{ t('home.navStats') }}</a>
        </div>

        <div class="dk-nav-actions">
          <LocaleSwitcher />
          <a
            v-if="docUrl"
            :href="docUrl"
            target="_blank"
            rel="noopener noreferrer"
            class="dk-icon-btn"
            :title="t('home.viewDocs')"
          >
            <Icon name="book" size="sm" :stroke-width="2" />
          </a>
          <router-link
            v-if="showModelPlazaEntry"
            to="/model-plaza"
            class="dk-icon-btn"
            :title="t('nav.modelPlaza')"
          >
            <Icon name="grid" size="sm" :stroke-width="2" />
          </router-link>
          <button
            type="button"
            class="dk-icon-btn"
            :title="isDark ? t('home.switchToLight') : t('home.switchToDark')"
            @click="toggleTheme"
          >
            <Icon v-if="isDark" name="sun" size="sm" :stroke-width="2" />
            <Icon v-else name="moon" size="sm" :stroke-width="2" />
          </button>

          <router-link v-if="isAuthenticated" :to="dashboardPath" class="dk-btn-sm">
            <span class="dk-avatar">{{ userInitial }}</span>
            {{ t('home.dashboard') }}
          </router-link>
          <router-link v-else to="/login" class="dk-btn-sm">{{ t('home.login') }}</router-link>
        </div>
      </nav>
    </header>

    <main class="dk-main">
      <!-- Hero + 控制台预览（合并为一屏，紧凑） -->
      <section class="dk-hero">
        <div class="dk-hero-copy">
          <span class="dk-pill">
            <span class="dk-pill-dot" aria-hidden="true"></span>
            {{ t('home.heroBadge') }}
          </span>

          <h1 class="dk-title">
            {{ siteName }}
          </h1>

          <p class="dk-subtitle">{{ siteSubtitle }}</p>

          <div class="dk-cta-row">
            <router-link :to="isAuthenticated ? dashboardPath : '/login'" class="dk-cta">
              {{ isAuthenticated ? t('home.goToDashboard') : t('home.getStarted') }}
              <Icon name="arrowRight" size="sm" :stroke-width="2.5" />
            </router-link>
            <a v-if="docUrl" :href="docUrl" target="_blank" rel="noopener noreferrer" class="dk-cta-ghost">
              <Icon name="book" size="sm" :stroke-width="2" />
              {{ t('home.viewDocs') }}
            </a>
          </div>

          <ul class="dk-tags">
            <li v-for="tag in featureTags" :key="tag.key" class="dk-tag">
              <Icon :name="tag.icon" size="xs" :stroke-width="2.5" />
              {{ t(tag.key) }}
            </li>
          </ul>
        </div>

        <!-- 控制台预览 -->
        <div class="terminal-container dk-hero-visual">
          <div class="dk-window">
            <div class="dk-window-bar">
              <span class="dk-dot dk-dot-r"></span>
              <span class="dk-dot dk-dot-y"></span>
              <span class="dk-dot dk-dot-g"></span>
              <span class="dk-window-title">{{ t('home.consoleTitle') }}</span>
              <span class="dk-live">
                <span class="dk-live-dot"></span>
                LIVE
              </span>
            </div>

            <div class="dk-window-body">
              <div class="dk-line dk-line-1">
                <span class="dk-k">POST</span>
                <span class="dk-u">/v1/messages</span>
                <span class="dk-m">204.4ms</span>
              </div>
              <div class="dk-line dk-line-2">
                <span class="dk-k">POST</span>
                <span class="dk-u">/v1/chat/completions</span>
                <span class="dk-m">176.2ms</span>
              </div>
              <div class="dk-line dk-line-3">
                <span class="dk-k">POST</span>
                <span class="dk-u">/v1/messages</span>
                <span class="dk-m">231.8ms</span>
              </div>
              <div class="dk-line dk-line-4">
                <span class="dk-ok">200</span>
                <span class="dk-u">stream closed</span>
                <span class="dk-m">1.8s</span>
              </div>
            </div>

            <div class="dk-window-foot">
              <span class="dk-foot-item">
                <i class="dk-foot-dot dk-dot-g2"></i>
                {{ t('home.floatLatency') }} <b>68ms</b>
              </span>
              <span class="dk-foot-item">
                <i class="dk-foot-dot dk-dot-b"></i>
                {{ t('home.floatUptime') }} <b>99.99%</b>
              </span>
              <span class="dk-foot-item">
                <i class="dk-foot-dot dk-dot-p"></i>
                {{ t('home.floatQps') }} <b>1.2K</b>
              </span>
            </div>
          </div>
        </div>
      </section>

      <!-- 数字横条（满宽，紧凑） -->
      <section id="stats" class="dk-stats">
        <div v-for="stat in stats" :key="stat.label" class="dk-stat">
          <span class="dk-stat-value" :class="stat.tone">{{ stat.value }}</span>
          <span class="dk-stat-label">{{ t(stat.label) }}</span>
        </div>
      </section>

      <!-- 特性：三列铺满 -->
      <section id="features" class="dk-section">
        <div class="dk-section-head">
          <h2 class="dk-section-title">{{ t('home.featuresTitle') }}</h2>
          <p class="dk-section-desc">{{ t('home.featuresSubtitle') }}</p>
        </div>

        <div class="dk-features">
          <article v-for="f in features" :key="f.title" class="dk-feature">
            <span class="dk-feature-glow" aria-hidden="true"></span>
            <span class="dk-feature-icon" :class="f.tone">
              <Icon :name="f.icon" size="md" :stroke-width="2" />
            </span>
            <h3 class="dk-feature-title">{{ t(f.title) }}</h3>
            <p class="dk-feature-desc">{{ t(f.desc) }}</p>
            <span class="dk-feature-index">{{ f.index }}</span>
          </article>
        </div>
      </section>

      <!-- 平台 + 能力清单 双栏 -->
      <section id="providers" class="dk-section">
        <div class="dk-split">
          <div>
            <h2 class="dk-section-title dk-section-title-sm">{{ t('home.providers.title') }}</h2>
            <p class="dk-section-desc">{{ t('home.providers.description') }}</p>

            <ul class="dk-providers">
              <li
                v-for="p in providers"
                :key="p.name"
                class="dk-provider"
                :class="{ 'dk-provider-soon': p.soon }"
              >
                <span class="dk-provider-logo" :style="{ background: p.bg }">{{ p.initial }}</span>
                <span class="dk-provider-name">{{ p.name }}</span>
                <span class="dk-provider-flag" :class="{ 'dk-provider-flag-soon': p.soon }">
                  {{ p.soon ? t('home.providers.soon') : '✓' }}
                </span>
              </li>
            </ul>
          </div>

          <ul class="dk-checklist">
            <li v-for="item in checklist" :key="item" class="dk-check-item">
              <span class="dk-check-mark">
                <Icon name="check" size="xs" :stroke-width="3" />
              </span>
              {{ t(item) }}
            </li>
          </ul>
        </div>
      </section>

      <!-- 底部 CTA -->
      <section class="dk-final">
        <span class="dk-final-beam" aria-hidden="true"></span>
        <h2 class="dk-final-title">{{ t('home.finalCtaTitle') }}</h2>
        <p class="dk-final-desc">{{ t('home.finalCtaDesc') }}</p>
        <router-link :to="isAuthenticated ? dashboardPath : '/login'" class="dk-cta dk-cta-lg">
          {{ isAuthenticated ? t('home.goToDashboard') : t('home.getStarted') }}
          <Icon name="arrowRight" size="sm" :stroke-width="2.5" />
        </router-link>
      </section>
    </main>

    <footer class="dk-footer">
      <span>&copy; {{ currentYear }} {{ siteName }}. {{ t('home.footer.allRightsReserved') }}</span>
      <div class="dk-footer-links">
        <a v-if="docUrl" :href="docUrl" target="_blank" rel="noopener noreferrer">{{ t('home.docs') }}</a>
        <a :href="githubUrl" target="_blank" rel="noopener noreferrer">GitHub</a>
      </div>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { useAuthStore, useAppStore } from '@/stores'
import LocaleSwitcher from '@/components/common/LocaleSwitcher.vue'
import Icon from '@/components/icons/Icon.vue'
import { sanitizeUrl } from '@/utils/url'
import { FeatureFlags, isFeatureFlagEnabled } from '@/utils/featureFlags'

type IconName = InstanceType<typeof Icon>['$props']['name']

const { t } = useI18n()

const authStore = useAuthStore()
const appStore = useAppStore()

const siteName = computed(
  () => appStore.cachedPublicSettings?.site_name || appStore.siteName || 'Sub2API'
)
const siteLogo = computed(() =>
  sanitizeUrl(appStore.cachedPublicSettings?.site_logo || appStore.siteLogo || '', {
    allowRelative: true,
    allowDataUrl: true
  })
)
const siteSubtitle = computed(
  () => appStore.cachedPublicSettings?.site_subtitle || 'AI API Gateway Platform'
)
const docUrl = computed(() =>
  sanitizeUrl(appStore.cachedPublicSettings?.doc_url || appStore.docUrl || '')
)
const homeContent = computed(() => appStore.cachedPublicSettings?.home_content || '')
const hasHomeContent = computed(() => homeContent.value.trim().length > 0)
const compactHomeEnabled = computed(
  () => appStore.cachedPublicSettings?.compact_home_enabled === true
)
const modelPlazaEnabled = computed(() => isFeatureFlagEnabled(FeatureFlags.modelPlaza))

const isHomeContentUrl = computed(() => {
  const content = homeContent.value.trim()
  return content.startsWith('http://') || content.startsWith('https://')
})

const isDark = ref(document.documentElement.classList.contains('dark'))
const githubUrl = 'https://github.com/Wei-Shaw/sub2api'

const isAuthenticated = computed(() => authStore.isAuthenticated)
const modelPlazaRequiresAuth = computed(
  () => appStore.cachedPublicSettings?.model_plaza_require_auth === true
)
const showModelPlazaEntry = computed(
  () => modelPlazaEnabled.value && (isAuthenticated.value || !modelPlazaRequiresAuth.value)
)
const isAdmin = computed(() => authStore.isAdmin)
const dashboardPath = computed(() => (isAdmin.value ? '/admin/dashboard' : '/dashboard'))
const userInitial = computed(() => {
  const user = authStore.user
  if (!user || !user.email) return ''
  return user.email.charAt(0).toUpperCase()
})
const currentYear = computed(() => new Date().getFullYear())

const featureTags = [
  { key: 'home.tags.subscriptionToApi', icon: 'swap' as IconName },
  { key: 'home.tags.stickySession', icon: 'shield' as IconName },
  { key: 'home.tags.realtimeBilling', icon: 'chart' as IconName }
]

/** 数字横条 */
const stats = [
  { value: '68ms', label: 'home.statLatency', tone: 'dk-tone-green' },
  { value: '99.99%', label: 'home.statUptime', tone: 'dk-tone-blue' },
  { value: '1.2K', label: 'home.statQps', tone: 'dk-tone-purple' },
  { value: '5', label: 'home.statPlatforms', tone: 'dk-tone-amber' }
]

const features = [
  {
    index: '01',
    icon: 'server' as IconName,
    title: 'home.features.unifiedGateway',
    desc: 'home.features.unifiedGatewayDesc',
    tone: 'dk-tone-green'
  },
  {
    index: '02',
    icon: 'users' as IconName,
    title: 'home.features.multiAccount',
    desc: 'home.features.multiAccountDesc',
    tone: 'dk-tone-blue'
  },
  {
    index: '03',
    icon: 'dollar' as IconName,
    title: 'home.features.balanceQuota',
    desc: 'home.features.balanceQuotaDesc',
    tone: 'dk-tone-purple'
  }
]

const providers = computed(() => [
  { name: t('home.providers.claude'), initial: 'C', bg: 'linear-gradient(135deg,#f97316,#ea580c)' },
  { name: 'GPT', initial: 'G', bg: 'linear-gradient(135deg,#22c55e,#15803d)' },
  { name: t('home.providers.gemini'), initial: 'G', bg: 'linear-gradient(135deg,#3b82f6,#1d4ed8)' },
  {
    name: t('home.providers.antigravity'),
    initial: 'A',
    bg: 'linear-gradient(135deg,#f43f5e,#be123c)'
  },
  {
    name: t('home.providers.more'),
    initial: '+',
    bg: 'linear-gradient(135deg,#475569,#334155)',
    soon: true
  }
])

const checklist = [
  'home.checklist.routing',
  'home.checklist.failover',
  'home.checklist.concurrency',
  'home.checklist.billing',
  'home.checklist.audit'
]

function toggleTheme() {
  isDark.value = !isDark.value
  document.documentElement.classList.toggle('dark', isDark.value)
  localStorage.setItem('theme', isDark.value ? 'dark' : 'light')
}

function initTheme() {
  const savedTheme = localStorage.getItem('theme')
  if (
    savedTheme === 'dark' ||
    (!savedTheme && window.matchMedia('(prefers-color-scheme: dark)').matches)
  ) {
    isDark.value = true
    document.documentElement.classList.add('dark')
  }
}

onMounted(() => {
  initTheme()
  authStore.checkAuth()
  if (!appStore.publicSettingsLoaded) {
    appStore.fetchPublicSettings()
  }
})
</script>

<style scoped>
/* ============================================================
   根：始终深色（科技风），与主题无关
   ============================================================ */
.dk-root {
  position: relative;
  min-height: 100vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  background: #05070d;
  color: #e2e8f0;
}

/* ============ 背景 ============ */
.dk-bg {
  position: absolute;
  inset: 0;
  pointer-events: none;
  overflow: hidden;
}

.dk-beam {
  position: absolute;
  border-radius: 9999px;
  filter: blur(110px);
  opacity: 0.5;
}

.dk-beam-a {
  top: -20rem;
  left: 50%;
  transform: translateX(-50%);
  height: 34rem;
  width: 56rem;
  background: radial-gradient(ellipse, rgba(20, 184, 166, 0.34) 0%, transparent 70%);
}

.dk-beam-b {
  bottom: -18rem;
  right: -10rem;
  height: 28rem;
  width: 34rem;
  background: radial-gradient(ellipse, rgba(59, 130, 246, 0.24) 0%, transparent 70%);
}

.dk-glow {
  position: absolute;
  top: 8rem;
  left: 12%;
  height: 18rem;
  width: 18rem;
  border-radius: 9999px;
  filter: blur(90px);
  background: radial-gradient(circle, rgba(168, 85, 247, 0.2) 0%, transparent 70%);
}

.dk-grid {
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(to right, rgba(148, 163, 184, 0.07) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(148, 163, 184, 0.07) 1px, transparent 1px);
  background-size: 44px 44px;
  -webkit-mask-image: radial-gradient(ellipse 85% 55% at 50% 0%, #000 20%, transparent 100%);
  mask-image: radial-gradient(ellipse 85% 55% at 50% 0%, #000 20%, transparent 100%);
}

.dk-noise {
  position: absolute;
  inset: 0;
  opacity: 0.035;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='120' height='120'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='3'/%3E%3C/filter%3E%3Crect width='120' height='120' filter='url(%23n)'/%3E%3C/svg%3E");
}

/* ============ 顶栏 ============ */
.dk-header {
  position: relative;
  z-index: 20;
  border-bottom: 1px solid rgba(148, 163, 184, 0.1);
  background: rgba(5, 7, 13, 0.7);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
}

.dk-nav {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  max-width: 78rem;
  margin: 0 auto;
  padding: 0.75rem 1.5rem;
}

.dk-brand {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  min-width: 0;
}

.dk-brand-logo {
  display: inline-flex;
  height: 1.75rem;
  width: 1.75rem;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  border-radius: 0.4375rem;
  background: rgba(15, 23, 42, 0.9);
  border: 1px solid rgba(45, 212, 191, 0.3);
  box-shadow: 0 0 16px rgba(45, 212, 191, 0.2);
}

.dk-brand-name {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  font-size: 0.9375rem;
  font-weight: 700;
  letter-spacing: -0.01em;
  color: #f8fafc;
}

/* 导航中间链接 */
.dk-nav-links {
  display: none;
  align-items: center;
  gap: 1.25rem;
  margin-left: 0.5rem;
}

@media (min-width: 900px) {
  .dk-nav-links {
    display: flex;
  }
}

.dk-nav-link {
  font-size: 0.8125rem;
  font-weight: 500;
  color: #94a3b8;
  transition: color 150ms ease;
}

.dk-nav-link:hover {
  color: #5eead4;
}

.dk-nav-actions {
  display: flex;
  align-items: center;
  gap: 0.375rem;
  margin-left: auto;
}

.dk-icon-btn {
  display: inline-flex;
  height: 1.875rem;
  width: 1.875rem;
  align-items: center;
  justify-content: center;
  border-radius: 0.4375rem;
  color: #94a3b8;
  transition: background-color 150ms ease, color 150ms ease;
}

.dk-icon-btn:hover {
  background: rgba(45, 212, 191, 0.1);
  color: #5eead4;
}

.dk-btn-sm {
  display: inline-flex;
  align-items: center;
  gap: 0.3125rem;
  border-radius: 0.4375rem;
  padding: 0.375rem 0.8125rem;
  font-size: 0.75rem;
  font-weight: 600;
  color: #05070d;
  background: #5eead4;
  box-shadow: 0 0 20px rgba(45, 212, 191, 0.4);
  transition: transform 150ms ease, box-shadow 150ms ease;
}

.dk-btn-sm:hover {
  transform: translateY(-1px);
  box-shadow: 0 0 28px rgba(45, 212, 191, 0.6);
}

.dk-avatar {
  display: inline-flex;
  height: 1.125rem;
  width: 1.125rem;
  align-items: center;
  justify-content: center;
  border-radius: 9999px;
  font-size: 0.5rem;
  font-weight: 800;
  background: rgba(5, 7, 13, 0.25);
}

/* ============ 主体 ============ */
.dk-main {
  position: relative;
  z-index: 10;
  flex: 1;
  width: 100%;
  max-width: 78rem;
  margin: 0 auto;
  padding: 0 1.5rem 2rem;
}

/* ============ Hero ============ */
.dk-hero {
  display: grid;
  grid-template-columns: 1fr;
  gap: 2rem;
  align-items: center;
  padding: 2.5rem 0 2rem;
}

@media (min-width: 1024px) {
  .dk-hero {
    grid-template-columns: minmax(0, 1.1fr) minmax(0, 1fr);
    gap: 3.5rem;
    padding: 3rem 0 2.5rem;
  }
}

.dk-hero-copy {
  min-width: 0;
  text-align: center;
}

@media (min-width: 1024px) {
  .dk-hero-copy {
    text-align: left;
  }
}

.dk-pill {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  border-radius: 9999px;
  padding: 0.1875rem 0.625rem;
  font-size: 0.6875rem;
  font-weight: 600;
  color: #5eead4;
  background: rgba(20, 184, 166, 0.1);
  border: 1px solid rgba(45, 212, 191, 0.25);
}

.dk-pill-dot {
  height: 0.375rem;
  width: 0.375rem;
  border-radius: 9999px;
  background: #2dd4bf;
  box-shadow: 0 0 0 3px rgba(45, 212, 191, 0.2);
  animation: dkPulse 2s ease-in-out infinite;
}

@keyframes dkPulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}

.dk-title {
  margin-top: 0.875rem;
  font-size: 2.5rem;
  line-height: 1.05;
  font-weight: 800;
  letter-spacing: -0.04em;
  color: #f8fafc;
}

@media (min-width: 768px) {
  .dk-title {
    font-size: 3.25rem;
  }
}

.dk-subtitle {
  margin-top: 0.75rem;
  max-width: 34rem;
  margin-left: auto;
  margin-right: auto;
  font-size: 0.9375rem;
  line-height: 1.65;
  color: #94a3b8;
}

@media (min-width: 1024px) {
  .dk-subtitle {
    margin-left: 0;
    margin-right: 0;
  }
}

.dk-cta-row {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: 0.625rem;
  margin-top: 1.5rem;
}

@media (min-width: 1024px) {
  .dk-cta-row {
    justify-content: flex-start;
  }
}

.dk-cta {
  display: inline-flex;
  align-items: center;
  gap: 0.4375rem;
  border-radius: 0.5rem;
  padding: 0.6875rem 1.25rem;
  font-size: 0.8125rem;
  font-weight: 600;
  color: #05070d;
  background: linear-gradient(135deg, #5eead4 0%, #2dd4bf 100%);
  box-shadow: 0 8px 24px -8px rgba(45, 212, 191, 0.7);
  transition: transform 160ms ease, box-shadow 160ms ease;
}

.dk-cta:hover {
  transform: translateY(-1px);
  box-shadow: 0 14px 32px -10px rgba(45, 212, 191, 0.85);
}

.dk-cta-lg {
  padding: 0.8125rem 1.625rem;
  font-size: 0.875rem;
}

.dk-cta-ghost {
  display: inline-flex;
  align-items: center;
  gap: 0.375rem;
  border-radius: 0.5rem;
  padding: 0.6875rem 1rem;
  font-size: 0.8125rem;
  font-weight: 500;
  color: #cbd5e1;
  background: rgba(30, 41, 59, 0.5);
  border: 1px solid rgba(148, 163, 184, 0.18);
  transition: background-color 150ms ease, border-color 150ms ease;
}

.dk-cta-ghost:hover {
  background: rgba(30, 41, 59, 0.8);
  border-color: rgba(45, 212, 191, 0.35);
}

.dk-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4375rem;
  margin-top: 1.5rem;
  justify-content: center;
}

@media (min-width: 1024px) {
  .dk-tags {
    justify-content: flex-start;
  }
}

.dk-tag {
  display: inline-flex;
  align-items: center;
  gap: 0.3125rem;
  border-radius: 0.375rem;
  padding: 0.25rem 0.5625rem;
  font-size: 0.6875rem;
  font-weight: 500;
  color: #94a3b8;
  background: rgba(30, 41, 59, 0.5);
  border: 1px solid rgba(148, 163, 184, 0.12);
}

/* ============ 控制台预览 ============ */
.dk-hero-visual {
  position: relative;
  display: flex;
  justify-content: center;
  min-width: 0;
}

.dk-window {
  width: 100%;
  max-width: 30rem;
  overflow: hidden;
  border-radius: 0.75rem;
  background: rgba(10, 15, 25, 0.85);
  border: 1px solid rgba(45, 212, 191, 0.18);
  box-shadow:
    0 30px 70px -30px rgba(0, 0, 0, 0.9),
    0 0 60px -20px rgba(45, 212, 191, 0.35);
  backdrop-filter: blur(10px);
}

.dk-window-bar {
  display: flex;
  align-items: center;
  gap: 0.375rem;
  padding: 0.5625rem 0.875rem;
  border-bottom: 1px solid rgba(148, 163, 184, 0.1);
  background: rgba(15, 23, 42, 0.6);
}

.dk-dot {
  height: 0.5rem;
  width: 0.5rem;
  border-radius: 9999px;
}

.dk-dot-r { background: #ef4444; }
.dk-dot-y { background: #eab308; }
.dk-dot-g { background: #22c55e; }

.dk-window-title {
  flex: 1;
  text-align: center;
  font-family: ui-monospace, monospace;
  font-size: 0.625rem;
  color: #64748b;
}

.dk-live {
  display: inline-flex;
  align-items: center;
  gap: 0.25rem;
  border-radius: 9999px;
  padding: 0.0625rem 0.375rem;
  font-size: 0.5rem;
  font-weight: 800;
  letter-spacing: 0.08em;
  color: #4ade80;
  background: rgba(34, 197, 94, 0.14);
}

.dk-live-dot {
  height: 0.25rem;
  width: 0.25rem;
  border-radius: 9999px;
  background: #4ade80;
  animation: dkPulse 1.6s ease-in-out infinite;
}

.dk-window-body {
  padding: 0.5rem 0;
  font-family: ui-monospace, 'Fira Code', monospace;
  font-size: 0.75rem;
}

.dk-line {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  padding: 0.3125rem 0.875rem;
  opacity: 0;
  animation: dkLineIn 0.45s ease forwards;
}

.dk-line:hover {
  background: rgba(45, 212, 191, 0.06);
}

.dk-line-1 { animation-delay: 0.2s; }
.dk-line-2 { animation-delay: 0.5s; }
.dk-line-3 { animation-delay: 0.8s; }
.dk-line-4 { animation-delay: 1.1s; }

@keyframes dkLineIn {
  from { opacity: 0; transform: translateX(-6px); }
  to { opacity: 1; transform: translateX(0); }
}

.dk-k {
  border-radius: 0.1875rem;
  padding: 0.0625rem 0.3125rem;
  font-size: 0.5625rem;
  font-weight: 800;
  color: #4ade80;
  background: rgba(34, 197, 94, 0.14);
}

.dk-u {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  color: #cbd5e1;
}

.dk-m {
  font-size: 0.6875rem;
  color: #64748b;
  font-variant-numeric: tabular-nums;
}

.dk-ok {
  border-radius: 0.1875rem;
  padding: 0.0625rem 0.3125rem;
  font-size: 0.5625rem;
  font-weight: 800;
  color: #38bdf8;
  background: rgba(56, 189, 248, 0.14);
}

.dk-window-foot {
  display: flex;
  flex-wrap: wrap;
  gap: 0.875rem;
  padding: 0.625rem 0.875rem;
  border-top: 1px solid rgba(148, 163, 184, 0.1);
  background: rgba(15, 23, 42, 0.5);
}

.dk-foot-item {
  display: inline-flex;
  align-items: center;
  gap: 0.3125rem;
  font-size: 0.625rem;
  color: #64748b;
}

.dk-foot-item b {
  font-variant-numeric: tabular-nums;
  color: #e2e8f0;
}

.dk-foot-dot {
  height: 0.3125rem;
  width: 0.3125rem;
  border-radius: 9999px;
  display: inline-block;
}

.dk-dot-g2 { background: #4ade80; }
.dk-dot-b { background: #38bdf8; }
.dk-dot-p { background: #c084fc; }

/* ============ 数字横条 ============ */
.dk-stats {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 1px;
  margin: 0.5rem 0 2.5rem;
  overflow: hidden;
  border-radius: 0.75rem;
  background: rgba(148, 163, 184, 0.12);
  border: 1px solid rgba(148, 163, 184, 0.12);
}

@media (min-width: 768px) {
  .dk-stats {
    grid-template-columns: repeat(4, minmax(0, 1fr));
  }
}

.dk-stat {
  display: flex;
  flex-direction: column;
  gap: 0.0625rem;
  padding: 1rem;
  background: rgba(10, 15, 25, 0.9);
  transition: background-color 180ms ease;
}

.dk-stat:hover {
  background: rgba(20, 30, 45, 0.9);
}

.dk-stat-value {
  font-size: 1.5rem;
  font-weight: 800;
  letter-spacing: -0.02em;
  font-variant-numeric: tabular-nums;
}

.dk-stat-label {
  font-size: 0.625rem;
  font-weight: 500;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: #64748b;
}

.dk-tone-green { color: #4ade80; }
.dk-tone-blue { color: #38bdf8; }
.dk-tone-purple { color: #c084fc; }
.dk-tone-amber { color: #fbbf24; }

/* ============ 区块 ============ */
.dk-section {
  margin-bottom: 2.5rem;
}

.dk-section-head {
  margin-bottom: 1.25rem;
}

.dk-section-title {
  font-size: 1.25rem;
  font-weight: 700;
  letter-spacing: -0.02em;
  color: #f8fafc;
}

.dk-section-title-sm {
  font-size: 1.0625rem;
}

.dk-section-desc {
  margin-top: 0.25rem;
  font-size: 0.8125rem;
  color: #64748b;
}

/* ============ 特性 ============ */
.dk-features {
  display: grid;
  grid-template-columns: 1fr;
  gap: 0.75rem;
}

@media (min-width: 768px) {
  .dk-features {
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }
}

.dk-feature {
  position: relative;
  overflow: hidden;
  border-radius: 0.75rem;
  padding: 1.125rem;
  background: rgba(10, 15, 25, 0.8);
  border: 1px solid rgba(148, 163, 184, 0.12);
  transition: border-color 220ms ease, transform 220ms ease;
}

.dk-feature:hover {
  transform: translateY(-2px);
  border-color: rgba(45, 212, 191, 0.35);
}

.dk-feature-glow {
  position: absolute;
  top: -3rem;
  right: -3rem;
  height: 8rem;
  width: 8rem;
  border-radius: 9999px;
  background: radial-gradient(circle, rgba(45, 212, 191, 0.16) 0%, transparent 70%);
  opacity: 0;
  transition: opacity 220ms ease;
  pointer-events: none;
}

.dk-feature:hover .dk-feature-glow {
  opacity: 1;
}

.dk-feature-icon {
  display: inline-flex;
  height: 2rem;
  width: 2rem;
  align-items: center;
  justify-content: center;
  border-radius: 0.5rem;
  margin-bottom: 0.75rem;
  background: rgba(30, 41, 59, 0.8);
  border: 1px solid rgba(148, 163, 184, 0.15);
}

.dk-tone-green.dk-feature-icon { color: #4ade80; border-color: rgba(74, 222, 128, 0.25); }
.dk-tone-blue.dk-feature-icon { color: #38bdf8; border-color: rgba(56, 189, 248, 0.25); }
.dk-tone-purple.dk-feature-icon { color: #c084fc; border-color: rgba(192, 132, 252, 0.25); }

.dk-feature-title {
  font-size: 0.875rem;
  font-weight: 700;
  color: #f1f5f9;
}

.dk-feature-desc {
  margin-top: 0.3125rem;
  font-size: 0.75rem;
  line-height: 1.6;
  color: #64748b;
}

.dk-feature-index {
  position: absolute;
  top: 0.875rem;
  right: 1rem;
  font-family: ui-monospace, monospace;
  font-size: 0.75rem;
  font-weight: 800;
  color: rgba(148, 163, 184, 0.14);
}

/* ============ 双栏 ============ */
.dk-split {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1.5rem;
}

@media (min-width: 900px) {
  .dk-split {
    grid-template-columns: minmax(0, 1.4fr) minmax(0, 1fr);
    gap: 2rem;
  }
}

/* 平台 */
.dk-providers {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4375rem;
  margin-top: 0.875rem;
}

.dk-provider {
  display: inline-flex;
  align-items: center;
  gap: 0.4375rem;
  border-radius: 0.5rem;
  padding: 0.4375rem 0.6875rem;
  background: rgba(10, 15, 25, 0.8);
  border: 1px solid rgba(148, 163, 184, 0.12);
  transition: border-color 180ms ease;
}

.dk-provider:hover {
  border-color: rgba(45, 212, 191, 0.35);
}

.dk-provider-soon {
  opacity: 0.55;
  border-style: dashed;
}

.dk-provider-logo {
  display: inline-flex;
  height: 1.375rem;
  width: 1.375rem;
  align-items: center;
  justify-content: center;
  border-radius: 0.3125rem;
  font-size: 0.5625rem;
  font-weight: 800;
  color: #ffffff;
}

.dk-provider-name {
  font-size: 0.75rem;
  font-weight: 600;
  color: #cbd5e1;
}

.dk-provider-flag {
  font-size: 0.625rem;
  font-weight: 800;
  color: #4ade80;
}

.dk-provider-flag-soon {
  color: #64748b;
}

/* 能力清单 */
.dk-checklist {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  padding: 0.875rem 1rem;
  border-radius: 0.75rem;
  background: rgba(10, 15, 25, 0.7);
  border: 1px solid rgba(148, 163, 184, 0.12);
}

@media (min-width: 900px) {
  .dk-checklist {
    margin-top: 1.875rem;
  }
}

.dk-check-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.75rem;
  color: #94a3b8;
}

.dk-check-mark {
  display: inline-flex;
  height: 1.125rem;
  width: 1.125rem;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  border-radius: 9999px;
  color: #4ade80;
  background: rgba(74, 222, 128, 0.12);
}

/* ============ 底部 CTA ============ */
.dk-final {
  position: relative;
  overflow: hidden;
  border-radius: 1rem;
  padding: 2.5rem 1.5rem;
  text-align: center;
  background: rgba(10, 15, 25, 0.85);
  border: 1px solid rgba(45, 212, 191, 0.22);
  box-shadow: 0 0 60px -24px rgba(45, 212, 191, 0.5);
}

.dk-final-beam {
  position: absolute;
  top: -6rem;
  left: 50%;
  height: 14rem;
  width: 30rem;
  transform: translateX(-50%);
  border-radius: 9999px;
  background: radial-gradient(ellipse, rgba(45, 212, 191, 0.22) 0%, transparent 70%);
  pointer-events: none;
}

.dk-final-title {
  position: relative;
  font-size: 1.375rem;
  font-weight: 800;
  letter-spacing: -0.02em;
  color: #f8fafc;
}

.dk-final-desc {
  position: relative;
  margin: 0.4375rem auto 1.25rem;
  max-width: 30rem;
  font-size: 0.8125rem;
  line-height: 1.6;
  color: #64748b;
}

.dk-final .dk-cta {
  position: relative;
}

/* ============ 页脚 ============ */
.dk-footer {
  position: relative;
  z-index: 10;
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  border-top: 1px solid rgba(148, 163, 184, 0.1);
  padding: 1.25rem 1.5rem;
  font-size: 0.6875rem;
  color: #475569;
}

@media (min-width: 640px) {
  .dk-footer {
    justify-content: space-between;
    max-width: 78rem;
    margin: 0 auto;
    width: 100%;
  }
}

.dk-footer-links {
  display: flex;
  gap: 1rem;
}

.dk-footer-links a {
  transition: color 150ms ease;
}

.dk-footer-links a:hover {
  color: #5eead4;
}

@media (prefers-reduced-motion: reduce) {
  .dk-line,
  .dk-pill-dot,
  .dk-live-dot {
    animation: none;
    opacity: 1;
  }
}
</style>
