<script setup lang="ts">
import { computed, markRaw, ref } from 'vue'
import DashboardShell from './components/DashboardShell.vue'
import CommunityManagement from './pages/CommunityManagement.vue'
import SecurityMonitoring from './pages/SecurityMonitoring.vue'
import CimPlatform from './pages/CimPlatform.vue'
import EnergyMonitoring from './pages/EnergyMonitoring.vue'
import EnergyAnalysis from './pages/EnergyAnalysis.vue'

type TabId =
  | 'community'
  | 'security'
  | 'cim'
  | 'energy-monitoring'
  | 'energy-analysis'

type DashboardTab = {
  id: TabId
  label: string
  component: unknown
}

const tabs: DashboardTab[] = [
  {
    id: 'community',
    label: '社区管理',
    component: markRaw(CommunityManagement),
  },
  {
    id: 'security',
    label: '安保监控',
    component: markRaw(SecurityMonitoring),
  },
  {
    id: 'cim',
    label: 'CIM平台',
    component: markRaw(CimPlatform),
  },
  {
    id: 'energy-monitoring',
    label: '能源检测',
    component: markRaw(EnergyMonitoring),
  },
  {
    id: 'energy-analysis',
    label: '节能分析',
    component: markRaw(EnergyAnalysis),
  },
]

const activeTab = ref<TabId>('community')

const activeComponent = computed(
  () => tabs.find((tab) => tab.id === activeTab.value)?.component ?? CommunityManagement,
)
</script>

<template>
  <DashboardShell v-model:active-tab="activeTab" :tabs="tabs">
    <component :is="activeComponent" />
  </DashboardShell>
</template>
