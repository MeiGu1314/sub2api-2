<template>
  <div class="qa-panel" :class="{ 'qa-panel-grid': variant === 'grid' }">
    <div v-if="variant !== 'grid'" class="mb-3 flex items-center justify-between">
      <h2 class="flex items-center gap-2 text-sm font-semibold text-gray-900 dark:text-white">
        <Icon name="bolt" size="xs" :stroke-width="2.5" class="text-primary-500" />
        {{ t('dashboard.quickActions') }}
      </h2>
    </div>

    <div class="space-y-2" :class="{ 'qa-grid': variant === 'grid' }">
      <button
        v-for="action in actions"
        :key="action.path"
        type="button"
        class="qa-item group"
        :class="`qa-${action.tone}`"
        @click="router.push(action.path)"
      >
        <span class="qa-icon">
          <Icon :name="action.icon" size="md" :stroke-width="2" />
        </span>
        <span class="min-w-0 flex-1 text-left">
          <span class="block truncate text-sm font-semibold text-gray-900 dark:text-white">
            {{ t(action.titleKey) }}
          </span>
          <span class="mt-0.5 block truncate text-xs text-gray-500 dark:text-dark-400">
            {{ t(action.descKey) }}
          </span>
        </span>
        <Icon name="chevronRight" size="sm" class="qa-chevron" />
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'
import Icon from '@/components/icons/Icon.vue'
import { useBatchImageAccess } from '@/composables/useBatchImageAccess'

const router = useRouter()
const { t } = useI18n()
const { canUseBatchImage, refreshBatchImageAccess } = useBatchImageAccess()

interface Props {
  /** grid：横向铺开（新用户场景），list：纵向列表（侧栏场景） */
  variant?: 'list' | 'grid'
}

withDefaults(defineProps<Props>(), {
  variant: 'list'
})

type IconName = InstanceType<typeof Icon>['$props']['name']

interface QuickAction {
  path: string
  icon: IconName
  titleKey: string
  descKey: string
  tone: 'teal' | 'emerald' | 'sky' | 'amber'
}

const actions = computed<QuickAction[]>(() => {
  const list: QuickAction[] = [
    {
      path: '/keys',
      icon: 'key',
      titleKey: 'dashboard.createApiKey',
      descKey: 'dashboard.generateNewKey',
      tone: 'teal'
    },
    {
      path: '/usage',
      icon: 'chart',
      titleKey: 'dashboard.viewUsage',
      descKey: 'dashboard.checkDetailedLogs',
      tone: 'emerald'
    }
  ]

  if (canUseBatchImage.value) {
    list.push({
      path: '/batch-image',
      icon: 'sparkles',
      titleKey: 'dashboard.batchImageAgent',
      descKey: 'dashboard.batchImageAgentDesc',
      tone: 'sky'
    })
  }

  list.push({
    path: '/redeem',
    icon: 'gift',
    titleKey: 'dashboard.redeemCode',
    descKey: 'dashboard.addBalanceWithCode',
    tone: 'amber'
  })

  return list
})

onMounted(() => {
  void refreshBatchImageAccess()
})
</script>

<style scoped>
.qa-panel {
  border-radius: 0.875rem;
  padding: 0.875rem;
  background: rgb(255 255 255 / 0.76);
  border: 1px solid rgb(255 255 255 / 0.7);
  box-shadow: 0 1px 2px rgba(15, 23, 42, 0.04);
  backdrop-filter: blur(14px) saturate(1.15);
  -webkit-backdrop-filter: blur(14px) saturate(1.15);
}

:root.dark .qa-panel {
  background: rgb(30 41 59 / 0.55);
  border-color: rgb(148 163 184 / 0.12);
}

.qa-item {
  display: flex;
  width: 100%;
  align-items: center;
  gap: 0.625rem;
  border-radius: 0.625rem;
  padding: 0.625rem;
  background: rgb(248 250 252 / 0.9);
  border: 1px solid rgb(226 232 240 / 0.8);
  transition: transform 170ms ease, border-color 170ms ease, box-shadow 170ms ease;
}

.qa-item:hover {
  transform: translateX(2px);
}

:root.dark .qa-item {
  background: rgb(15 23 42 / 0.4);
  border-color: rgb(51 65 85 / 0.7);
}

.qa-teal:hover {
  border-color: rgb(20 184 166 / 0.4);
  box-shadow: 0 6px 18px -12px rgba(13, 148, 136, 0.6);
}

.qa-emerald:hover {
  border-color: rgb(16 185 129 / 0.4);
  box-shadow: 0 6px 18px -12px rgba(5, 150, 105, 0.6);
}

.qa-sky:hover {
  border-color: rgb(14 165 233 / 0.4);
  box-shadow: 0 6px 18px -12px rgba(2, 132, 199, 0.6);
}

.qa-amber:hover {
  border-color: rgb(245 158 11 / 0.4);
  box-shadow: 0 6px 18px -12px rgba(217, 119, 6, 0.6);
}

.qa-icon {
  display: inline-flex;
  height: 2rem;
  width: 2rem;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  border-radius: 0.5rem;
}

.qa-teal .qa-icon { color: #0d9488; background: rgb(204 251 241); }
.qa-emerald .qa-icon { color: #059669; background: rgb(209 250 229); }
.qa-sky .qa-icon { color: #0284c7; background: rgb(224 242 254); }
.qa-amber .qa-icon { color: #d97706; background: rgb(254 243 199); }

:root.dark .qa-icon {
  background: rgb(15 23 42 / 0.65);
}

.qa-chevron {
  flex-shrink: 0;
  color: rgb(148 163 184);
  transition: transform 170ms ease, color 170ms ease;
}

.qa-item:hover .qa-chevron {
  transform: translateX(1px);
  color: rgb(13 148 136);
}

:root.dark .qa-item:hover .qa-chevron {
  color: rgb(94 234 212);
}

/* ============ grid 变体 ============ */
.qa-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 0.5rem;
}

@media (min-width: 640px) {
  .qa-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (min-width: 1280px) {
  .qa-grid {
    grid-template-columns: repeat(4, minmax(0, 1fr));
  }
}

.qa-grid > .qa-item {
  margin: 0;
}
</style>
