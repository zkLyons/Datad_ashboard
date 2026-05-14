<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from "vue";
import BottomNav from "./BottomNav.vue";

type Tab = {
  id: string;
  label: string;
};

const props = defineProps<{
  tabs: Tab[];
  activeTab: string;
}>();

const emit = defineEmits<{
  (event: "update:activeTab", value: string): void;
}>();

const now = ref(new Date());
let timer: number | undefined;

const weekdays = [
  "星期日",
  "星期一",
  "星期二",
  "星期三",
  "星期四",
  "星期五",
  "星期六",
];

const pad = (value: number) => value.toString().padStart(2, "0");

const leftClock = computed(() => {
  const date = now.value;
  return `${date.getFullYear()}/${pad(date.getMonth() + 1)}/${pad(date.getDate())}    ${pad(date.getHours())}:${pad(date.getMinutes())}:${pad(date.getSeconds())}`;
});

const rightClock = computed(() => {
  const date = now.value;
  return `${weekdays[date.getDay()]}  ${pad(date.getHours())}:${pad(date.getMinutes())}:${pad(date.getSeconds())}  |  ${date.getFullYear()}/${pad(date.getMonth() + 1)}/${pad(date.getDate())}`;
});

onMounted(() => {
  timer = window.setInterval(() => {
    now.value = new Date();
  }, 1000);
});

onBeforeUnmount(() => {
  if (timer) {
    window.clearInterval(timer);
  }
});
</script>

<template>
  <div class="dashboard-shell">
    <div class="dashboard-shell__overlay"></div>
    <header class="dashboard-shell__header">
      <div class="dashboard-shell__clock dashboard-shell__clock--left">
        {{ leftClock }}
      </div>
      <div class="dashboard-shell__clock dashboard-shell__clock--right">
        {{ rightClock }}
      </div>
    </header>

    <main class="dashboard-shell__content">
      <slot />
    </main>

    <footer class="dashboard-shell__footer">
      <BottomNav
        :tabs="props.tabs"
        :active-tab="props.activeTab"
        @update:active-tab="emit('update:activeTab', $event)"
      />
      <div class="dashboard-shell__chevron"></div>
    </footer>
  </div>
</template>

<style scoped>
.dashboard-shell {
  position: relative;
  min-height: 100vh;
  overflow: hidden;
  background: url("../assets/main.jpg") center/cover no-repeat;
}

.dashboard-shell__overlay {
  position: absolute;
  inset: 0;
  background:
    radial-gradient(
      ellipse 48% 36% at 50% 44%,
      rgba(255, 255, 255, 0.14) 0%,
      rgba(255, 255, 255, 0.06) 26%,
      rgba(10, 20, 42, 0.1) 50%,
      rgba(5, 12, 28, 0.42) 74%,
      rgba(2, 6, 16, 0.8) 100%
    ),
    linear-gradient(
      90deg,
      rgba(4, 13, 30, 0.84) 0,
      rgba(4, 13, 30, 0.35) 22%,
      rgba(4, 13, 30, 0.12) 50%,
      rgba(4, 13, 30, 0.35) 78%,
      rgba(4, 13, 30, 0.84) 100%
    ),
    linear-gradient(
      180deg,
      rgba(1, 6, 17, 0.6),
      rgba(1, 6, 17, 0.14) 14%,
      rgba(1, 6, 17, 0.1) 76%,
      rgba(1, 6, 17, 0.82)
    );
  pointer-events: none;
}

.dashboard-shell__header,
.dashboard-shell__content,
.dashboard-shell__footer {
  position: relative;
  z-index: 1;
}

.dashboard-shell__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 28px 0;
  color: rgba(242, 248, 255, 0.88);
  font-size: 14px;
  letter-spacing: 1px;
  text-shadow: 0 0 10px rgba(5, 10, 24, 0.7);
}

.dashboard-shell__clock {
  white-space: pre-wrap;
  opacity: 0.82;
}

.dashboard-shell__content {
  height: calc(100vh - 42px);
  padding: 12px 22px 92px;
  box-sizing: border-box;
  overflow: hidden;
}

.dashboard-shell__footer {
  position: absolute;
  left: 50%;
  bottom: 18px;
  transform: translateX(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.dashboard-shell__chevron {
  width: 20px;
  height: 12px;
  position: relative;
}

.dashboard-shell__chevron::before,
.dashboard-shell__chevron::after {
  content: "";
  position: absolute;
  bottom: 0;
  width: 12px;
  height: 2px;
  background: rgba(224, 236, 255, 0.9);
}

.dashboard-shell__chevron::before {
  left: 0;
  transform: rotate(35deg);
  transform-origin: left center;
}

.dashboard-shell__chevron::after {
  right: 0;
  transform: rotate(-35deg);
  transform-origin: right center;
}

@media (max-width: 1200px) {
  .dashboard-shell__header {
    padding: 10px 16px 0;
    font-size: 12px;
  }

  .dashboard-shell__content {
    padding: 12px 14px 92px;
  }
}
</style>
