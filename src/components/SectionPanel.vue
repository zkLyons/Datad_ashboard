<script setup lang="ts">
import titleBars from "../assets/png/img/titleImg.png";

defineProps<{
  title: string;
}>();
</script>

<template>
  <section class="section-panel">
    <header class="section-panel__header">
      <img :src="titleBars" alt="" class="section-panel__icon" />
      <h3>{{ title }}</h3>
    </header>
    <div class="section-panel__body">
      <slot></slot>
    </div>
  </section>
</template>

<style scoped>
.section-panel {
  position: relative;
  display: flex;
  flex-direction: column;
  min-height: 0;
  height: 100%;
  overflow: hidden;

  box-shadow:
    inset 0 1px 0 rgba(226, 243, 255, 0.1),
    inset 0 0 28px rgba(86, 154, 255, 0.05),
    0 12px 32px rgba(2, 8, 22, 0.12);
}

.section-panel__header {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 18px 18px 10px;
}

.section-panel__header h3 {
  margin: 0;
  color: #5c8fff;
  font-size: 15px;
  font-weight: 500;
  letter-spacing: 1px;
}

.section-panel__icon {
  width: 17px;
  height: 17px;
  object-fit: contain;
}

.section-panel__body {
  position: relative;
  display: flex;
  flex: 1;
  flex-direction: column;
  min-height: 0;
  padding: 0 18px 18px;
  z-index: 1;
}
</style>
