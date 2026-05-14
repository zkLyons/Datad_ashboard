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

  /* 边框稍微保留一点，以免卡片完全融入背景失去轮廓 */

  /* 核心修改 1：极限压低背景颜色透明度，几乎全透 */
  background: linear-gradient(
    180deg,
    rgba(9, 21, 51, 0.01),
    /* 顶部几乎完全透明 (0.02) */ rgba(232, 235, 240, 0.1)
      /* 底部保留极少的暗色压底 (0.1) */
  );

  box-shadow:
    inset 0 0 0 1px rgba(212, 214, 218, 0.02),
    0 8px 16px rgba(0, 0, 0, 0.05);
}

.section-panel::before {
  content: "";
  position: absolute;
  inset: 0;
  /* 伪元素高光同样降到极低，只留一丝光泽 */
  background:
    linear-gradient(
      90deg,
      rgba(89, 143, 255, 0.03),
      transparent 28%,
      transparent 72%,
      rgba(89, 143, 255, 0.03)
    ),
    linear-gradient(180deg, rgba(160, 212, 255, 0.02), transparent 18%);
  pointer-events: none;
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
