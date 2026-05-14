<script setup lang="ts">
defineProps<{
  title: string
  subtitle: string
}>()
</script>

<template>
  <section class="placeholder-page">
    <div class="placeholder-page__box">
      <p class="placeholder-page__eyebrow">智慧社区数字大屏</p>
      <h2>{{ title }}</h2>
      <p class="placeholder-page__subtitle">{{ subtitle }}</p>
    </div>
  </section>
</template>

<style scoped>
.placeholder-page {
  min-height: calc(100vh - 150px);
  display: grid;
  place-items: center;
}

.placeholder-page__box {
  width: min(720px, 88vw);
  padding: 56px 40px;
  text-align: center;
  border: 1px solid rgba(81, 122, 215, 0.28);
  background:
    linear-gradient(180deg, rgba(9, 20, 46, 0.72), rgba(6, 15, 38, 0.82));
  box-shadow:
    inset 0 0 0 1px rgba(119, 180, 255, 0.08),
    0 24px 40px rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(8px);
}

.placeholder-page__eyebrow {
  margin: 0 0 14px;
  color: rgba(123, 177, 255, 0.8);
  font-size: 14px;
  letter-spacing: 4px;
  text-transform: uppercase;
}

.placeholder-page__box h2 {
  margin: 0;
  color: #ffffff;
  font-weight: 500;
  font-size: clamp(34px, 4vw, 48px);
  letter-spacing: 2px;
}

.placeholder-page__subtitle {
  margin: 18px auto 0;
  max-width: 520px;
  color: rgba(226, 236, 255, 0.7);
  font-size: 16px;
  line-height: 1.8;
}
</style>
