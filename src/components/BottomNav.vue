<script setup lang="ts">
type Tab = {
  id: string
  label: string
}

const props = defineProps<{
  tabs: Tab[]
  activeTab: string
}>()

const emit = defineEmits<{
  (event: 'update:activeTab', value: string): void
}>()

const getButtonClass = (index: number) => {
  const centerIndex = Math.floor(props.tabs.length / 2)

  if (index === centerIndex) return 'nav-button--center'
  if (index < centerIndex) return 'nav-button--left'
  return 'nav-button--right'
}
</script>

<template>
  <nav class="bottom-nav" aria-label="页面切换">
    <button
      v-for="(tab, index) in tabs"
      :key="tab.id"
      type="button"
      class="nav-button"
      :class="[getButtonClass(index), { 'is-active': tab.id === activeTab }]"
      @click="emit('update:activeTab', tab.id)"
    >
      <span>{{ tab.label }}</span>
    </button>
  </nav>
</template>

<style scoped>
.bottom-nav {
  display: flex;
  justify-content: center;
  align-items: end;
  gap: 10px;
  flex-wrap: wrap;
  pointer-events: auto;
}

.nav-button {
  position: relative;
  height: 36px;
  padding: 0 22px;
  border: 0;
  background: transparent;
  color: #ffffff;
  font-size: 13px;
  letter-spacing: 1px;
  cursor: pointer;
  transition:
    transform 0.2s ease,
    color 0.2s ease,
    filter 0.2s ease;
}

.nav-button span {
  position: relative;
  z-index: 2;
}

.nav-button::before,
.nav-button::after {
  content: '';
  position: absolute;
  inset: 0;
  transition:
    background 0.2s ease,
    box-shadow 0.2s ease;
}

.nav-button::before {
  z-index: 0;
  background: linear-gradient(180deg, rgba(95, 110, 154, 0.28), rgba(47, 54, 73, 0.42));
  box-shadow:
    inset 0 0 0 1px rgba(93, 112, 233, 0.95),
    0 0 10px rgba(47, 73, 176, 0.18);
}

.nav-button::after {
  z-index: 1;
  inset: 3px;
  background: linear-gradient(180deg, rgba(126, 130, 138, 0.16), rgba(78, 81, 88, 0.3));
  opacity: 0.95;
}

.nav-button--left {
  min-width: 102px;
}

.nav-button--left::before,
.nav-button--left::after {
  clip-path: polygon(16px 0, 100% 0, calc(100% - 16px) 100%, 0 100%);
}

.nav-button--center {
  min-width: 120px;
  height: 42px;
  padding: 0 26px;
  font-size: 14px;
}

.nav-button--center::before,
.nav-button--center::after {
  clip-path: polygon(8px 0, calc(100% - 8px) 0, 100% 100%, 0 100%);
}

.nav-button--center::before {
  background: linear-gradient(180deg, rgba(111, 116, 128, 0.34), rgba(59, 61, 70, 0.5));
}

.nav-button--center::after {
  background: linear-gradient(180deg, rgba(133, 135, 141, 0.16), rgba(78, 80, 86, 0.28));
}

.nav-button--right {
  min-width: 102px;
}

.nav-button--right::before,
.nav-button--right::after {
  clip-path: polygon(0 0, calc(100% - 16px) 0, 100% 100%, 16px 100%);
}

.nav-button:hover {
  transform: translateY(-1px);
  filter: brightness(1.08);
}

.nav-button.is-active {
  color: #50fef0;
}

.nav-button.is-active::before {
  box-shadow:
    inset 0 0 0 1px rgba(92, 112, 245, 1),
    0 0 14px rgba(49, 92, 220, 0.3);
}

.nav-button.is-active::after {
  background: linear-gradient(180deg, rgba(56, 93, 185, 0.22), rgba(31, 44, 89, 0.44));
}
</style>
