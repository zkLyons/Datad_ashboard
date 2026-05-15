<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from "vue";
import { PieChart } from "echarts/charts";
import {
  GraphicComponent,
  TooltipComponent,
  type GraphicComponentOption,
  type TooltipComponentOption,
} from "echarts/components";
import { init, use, type ComposeOption, type ECharts } from "echarts/core";
import type { PieSeriesOption } from "echarts/charts";
import { CanvasRenderer } from "echarts/renderers";
import SectionPanel from "../components/SectionPanel.vue";
import overviewIcon from "../assets/png/img/title.png";
import faceIcon from "../assets/png/img/人.png";
import controlIcon from "../assets/png/img/控制中心.png";
import warningIcon from "../assets/png/img/警告.png";
import videoDemo from "../assets/png/img/videoDemo.mp4";

use([PieChart, GraphicComponent, TooltipComponent, CanvasRenderer]);

type SecurityChartOption = ComposeOption<
  GraphicComponentOption | TooltipComponentOption | PieSeriesOption
>;

const cameraStats = [
  { label: "人脸识别摄像头", value: "8", icon: faceIcon },
  { label: "可控制摄像头", value: "20", icon: controlIcon },
  { label: "故障摄像头", value: "2", icon: warningIcon },
];

const coverageMetrics = [
  { label: "人脸识别区域", value: 32, accent: "#4ee7d4" },
  { label: "可控摄像头区域", value: 53.685, accent: "#35d8b9" },
];

const alarms = [
  { name: "监控1: 大西门云台", time: "07:12:18", status: "待派遣" },
  { name: "监控2: A楼南侧", time: "07:13:12", status: "处理中" },
  { name: "监控3: T9北侧", time: "08:12:18", status: "已消警" },
  { name: "监控4: 大西门云台", time: "07:12:18", status: "待派遣" },
  { name: "监控5: A楼南侧", time: "07:13:12", status: "处理中" },
  { name: "监控6: T9北侧", time: "08:12:18", status: "已消警" },
];

const cameraFeeds = [
  { title: "星光广场云台", source: videoDemo, poster: "", muted: true },
  { title: "底商北侧", source: videoDemo, poster: "", muted: true },

  { title: "后山艺术空间", source: videoDemo, poster: "", muted: true },
  { title: "T9后面", source: videoDemo, poster: "", muted: true },
  { title: "门口", source: videoDemo, poster: "", muted: true },
  { title: "T9北侧", source: videoDemo, poster: "", muted: true },
  { title: "T9北侧", source: videoDemo, poster: "", muted: true },
  { title: "T9北侧", source: videoDemo, poster: "", muted: true },
];

const chartRef = ref<HTMLDivElement | null>(null);
let coverageChart: ECharts | null = null;

const totalCoverage = computed(() => 80);

const statusClassMap: Record<string, string> = {
  待派遣: "pending",
  处理中: "processing",
  已消警: "resolved",
};

const coverageSegments = [
  { value: totalCoverage.value, color: "#39dfd1" },
  { value: 100 - totalCoverage.value, color: "rgba(75, 103, 140, 0.34)" },
];

const renderCoverageChart = () => {
  if (!chartRef.value) {
    return;
  }

  coverageChart ??= init(chartRef.value);

  const option: SecurityChartOption = {
    animation: false,
    tooltip: { trigger: "item" },
    graphic: [
      {
        type: "text",
        left: "center",
        top: "36%",
        silent: true,
        style: {
          text: "覆盖区域",
          fill: "#63d0ff",
          fontSize: 13,
          fontFamily: "Microsoft YaHei, PingFang SC, sans-serif",
          align: "center",
        },
      },
      {
        type: "text",
        left: "center",
        top: "50%",
        silent: true,
        style: {
          text: `${totalCoverage.value}%`,
          fill: "#ffffff",
          fontSize: 18,
          fontWeight: 700,
          fontFamily: "Microsoft YaHei, PingFang SC, sans-serif",
          align: "center",
        },
      },
    ],
    series: [
      {
        type: "pie",
        radius: ["72%", "90%"],
        center: ["50%", "50%"],
        startAngle: 120,
        label: { show: false },
        labelLine: { show: false },
        itemStyle: { borderWidth: 0 },
        emphasis: { scale: false },
        data: coverageSegments.map((item) => ({
          value: item.value,
          itemStyle: { color: item.color },
        })),
      },
    ],
  };

  coverageChart.setOption(option);
};

const handleResize = () => {
  coverageChart?.resize();
};

onMounted(() => {
  renderCoverageChart();
  window.addEventListener("resize", handleResize);
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", handleResize);
  coverageChart?.dispose();
  coverageChart = null;
});
</script>

<template>
  <section class="security-page">
    <div class="security-page__side security-page__side--left">
      <SectionPanel title="摄像头概况总览" class="security-overview-panel">
        <div class="security-overview">
          <div class="security-overview__header">
            <img :src="overviewIcon" alt="" class="security-overview__icon" />
            <div class="security-overview__summary">
              <p class="security-overview__label">摄像头总数量</p>
              <strong>53</strong>
            </div>
          </div>

          <div class="security-overview__cards">
            <article
              v-for="item in cameraStats"
              :key="item.label"
              class="security-overview__card"
            >
              <span class="security-overview__card-label">
                <img :src="item.icon" alt="" class="security-overview__card-icon" />
                <span>{{ item.label }}</span>
              </span>
              <strong>{{ item.value }}</strong>
            </article>
          </div>
        </div>
      </SectionPanel>

      <SectionPanel title="摄像头覆盖占比" class="security-coverage-panel">
        <div class="coverage-panel">
          <div class="coverage-panel__chart-wrap">
            <div ref="chartRef" class="coverage-panel__chart"></div>
          </div>

          <div class="coverage-panel__metrics">
            <div class="coverage-panel__toggle">
              <span>覆盖区域显示</span>
              <button
                type="button"
                class="coverage-panel__switch is-active"
                aria-label="覆盖区域显示已开启"
              >
                <span></span>
              </button>
            </div>

            <div
              v-for="item in coverageMetrics"
              :key="item.label"
              class="coverage-panel__metric"
            >
              <div class="coverage-panel__metric-row">
                <span>{{ item.label }}: {{ item.value }}%</span>
              </div>
              <div class="coverage-panel__track">
                <div
                  class="coverage-panel__fill"
                  :style="{ width: `${item.value}%`, background: item.accent }"
                ></div>
              </div>
              <div class="coverage-panel__percent">{{ item.value }}%</div>
            </div>
          </div>
        </div>
      </SectionPanel>

      <SectionPanel title="报警讯息列表" class="security-alarm-panel">
        <div class="alarm-list">
          <article
            v-for="item in alarms"
            :key="`${item.name}-${item.time}`"
            class="alarm-item"
          >
            <div class="alarm-item__bar"></div>
            <div class="alarm-item__content">
              <div class="alarm-item__row alarm-item__row--main">
                <span class="alarm-item__name">{{ item.name }}</span>
                <span
                  class="alarm-item__status"
                  :class="statusClassMap[item.status]"
                >
                  {{ item.status }}
                </span>
              </div>
              <div class="alarm-item__row alarm-item__row--time">
                <div class="alarm-item__time">{{ item.time }}</div>
              </div>
            </div>
          </article>
        </div>
      </SectionPanel>
    </div>

    <div class="security-page__spacer" aria-hidden="true"></div>

    <div class="security-page__side security-page__side--right">
      <SectionPanel title="摄像头视频监控" class="video-panel">
        <div class="video-grid">
          <article
            v-for="feed in cameraFeeds"
            :key="`${feed.title}-${feed.source || 'empty'}`"
            class="video-card"
          >
            <div
              class="video-card__frame"
              :class="{ 'is-empty': !feed.source }"
            >
              <video
                v-if="feed.source"
                class="video-card__media"
                :src="feed.source"
                :poster="feed.poster"
                autoplay
                loop
                muted
                playsinline
              ></video>

              <div v-else class="video-card__placeholder">
                <div class="video-card__play"></div>
                <div class="video-card__line"></div>
                <div class="video-card__dots"></div>
              </div>
            </div>

            <div class="video-card__title">{{ feed.title }}</div>
          </article>
        </div>
      </SectionPanel>
    </div>
  </section>
</template>

<style scoped>
.security-page {
  height: 100%;
  min-height: 0;
  display: grid;
  grid-template-columns: 360px minmax(0, 1fr) 500px;
  gap: 14px;
}

.security-page__side {
  display: flex;
  flex-direction: column;
  min-height: 0;
  gap: 14px;
}

.security-page__side--right {
  max-width: 500px;
  width: 100%;
  justify-self: end;
}

.security-page__spacer {
  min-width: 0;
}

.security-overview-panel {
  height: 20%;
}

.security-coverage-panel {
  height: 22%;
}

.security-alarm-panel {
  flex: 1;
  min-height: 0;
}

.video-panel {
  flex: 1;
  min-height: 0;
}

:deep(.security-overview-panel .section-panel__header),
:deep(.security-coverage-panel .section-panel__header),
:deep(.security-alarm-panel .section-panel__header),
:deep(.video-panel .section-panel__header) {
  padding-bottom: 6px;
}

:deep(.security-overview-panel .section-panel__body),
:deep(.security-coverage-panel .section-panel__body),
:deep(.video-panel .section-panel__body) {
  padding-top: 0;
}

.security-overview {
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.security-overview__header {
  display: grid;
  grid-template-columns: 78px minmax(0, 1fr);
  align-items: center;
  column-gap: 16px;
}

.security-overview__icon {
  width: 52px;
  height: 52px;
  justify-self: center;
  object-fit: contain;
}

.security-overview__summary {
  /* min-width: 0; */
  width: 80px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  /* background-color: red; */
}

.security-overview__label {
  margin: 0 0 4px;
  color: #79a5ff;
  font-size: 13px;
  white-space: nowrap;
}

.security-overview__summary strong {
  color: #5ec8ff;
  font-size: 26px;
  font-weight: 500;
  line-height: 1.05;
}

.security-overview__cards {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 10px;
}

.security-overview__card {
  text-align: center;
}

.security-overview__card-label {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 4px 6px;
  margin-bottom: 6px;
  color: #ffffff;
  font-size: 12px;
  line-height: 1.1;
  white-space: nowrap;
  background: linear-gradient(
    90deg,
    rgba(18, 138, 204, 0.95),
    rgba(109, 85, 213, 0.95)
  );
}

.security-overview__card-label span {
  display: block;
}

.security-overview__card-icon {
  width: 12px;
  height: 12px;
  flex: 0 0 auto;
  object-fit: contain;
}

.security-overview__card strong {
  color: #e6f0ff;
  font-size: 17px;
  font-weight: 500;
}

.coverage-panel {
  display: grid;
  grid-template-columns: 150px minmax(0, 1fr);
  align-items: center;
  column-gap: 18px;
}

.coverage-panel__chart-wrap {
  width: 130px;
  height: 130px;
  margin: 0 auto;
}

.coverage-panel__chart {
  width: 130px;
  height: 130px;
}

.coverage-panel__metrics {
  display: flex;
  flex-direction: column;
  gap: 10px;
  min-width: 0;
}

.coverage-panel__toggle {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  color: #ffffff;
  font-size: 13px;
}

.coverage-panel__switch {
  position: relative;
  width: 38px;
  height: 22px;
  padding: 0;
  border: 0;
  border-radius: 999px;
  background: rgba(85, 98, 143, 0.95);
  cursor: default;
}

.coverage-panel__switch span {
  position: absolute;
  top: 2px;
  right: 2px;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #ffffff;
}

.coverage-panel__metric {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.coverage-panel__metric-row {
  color: #ffffff;
  font-size: 13px;
}

.coverage-panel__track {
  width: 100%;
  height: 15px;
  overflow: hidden;
  background: transparent;
}

.coverage-panel__fill {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  height: 100%;
  color: #ffffff;
  font-size: 11px;
  padding-right: 6px;
  transition: width 0.2s ease;
}

.coverage-panel__percent {
  align-self: flex-end;
  color: #ffffff;
  font-size: 11px;
}

.alarm-list {
  display: flex;
  flex: 1;
  flex-direction: column;
  gap: 8px;
  min-height: 0;
  overflow-y: auto;
  padding-right: 8px;
  scrollbar-width: thin;
  scrollbar-color: rgba(91, 232, 255, 0.92) rgba(255, 255, 255, 0.1);
}

.alarm-list::-webkit-scrollbar {
  width: 6px;
}

.alarm-list::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.08);
  border-radius: 999px;
}

.alarm-list::-webkit-scrollbar-thumb {
  border-radius: 999px;
  background: linear-gradient(
    180deg,
    rgba(83, 233, 255, 0.95),
    rgba(61, 117, 255, 0.95)
  );
}

.alarm-item {
  display: flex;
  gap: 14px;
  min-height: 52px;
}

.alarm-item__bar {
  width: 16px;
  background: linear-gradient(
    180deg,
    rgba(27, 117, 221, 0.88),
    rgba(30, 104, 182, 0.64)
  );
}

.alarm-item__content {
  flex: 1;
  padding-top: 2px;
}

.alarm-item__row {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  align-items: center;
}

.alarm-item__row--main {
  min-height: 28px;
  padding: 0 10px 0 0;
  background: rgba(10, 24, 54, 0.62);
}

.alarm-item__row--time {
  min-height: 24px;
  padding-top: 6px;
}

.alarm-item__name,
.alarm-item__time {
  color: #ffffff;
  font-size: 13px;
}

.alarm-item__time {
  color: rgba(255, 255, 255, 0.96);
}

.alarm-item__status {
  flex-shrink: 0;
  min-width: 48px;
  padding: 3px 8px;
  border-radius: 4px;
  color: #ffffff;
  font-size: 12px;
  text-align: center;
}

.alarm-item__status.pending {
  background: #7340d6;
}

.alarm-item__status.processing {
  background: #4b98ff;
}

.alarm-item__status.resolved {
  background: #345fd3;
}

.video-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 16px 14px;
  min-height: 0;
  max-height: 100%;
  overflow-y: auto;
  padding-right: 6px;
  padding-bottom: 4px;
  align-content: start;
  scrollbar-width: thin;
  scrollbar-color: rgba(91, 232, 255, 0.92) rgba(255, 255, 255, 0.1);
}

.video-grid::-webkit-scrollbar {
  width: 6px;
}

.video-grid::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.08);
  border-radius: 999px;
}

.video-grid::-webkit-scrollbar-thumb {
  border-radius: 999px;
  background: linear-gradient(
    180deg,
    rgba(83, 233, 255, 0.95),
    rgba(61, 117, 255, 0.95)
  );
}

.video-card {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.video-card__frame {
  position: relative;
  aspect-ratio: auto;
  min-height: 150px;
  overflow: hidden;
  background: linear-gradient(
    180deg,
    rgba(54, 54, 54, 0.92),
    rgba(8, 8, 8, 0.98)
  );
  box-shadow: inset 0 0 0 1px rgba(255, 255, 255, 0.03);
}

.video-card__frame::after {
  content: "";
  position: absolute;
  left: 16px;
  right: 18px;
  bottom: 20px;
  height: 4px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.38);
}

.video-card__media,
.video-card__placeholder {
  width: 100%;
  height: 100%;
}

.video-card__media {
  display: block;
  object-fit: cover;
}

.video-card__placeholder {
  position: relative;
  background: linear-gradient(
    180deg,
    rgba(64, 64, 64, 0.86),
    rgba(16, 16, 16, 0.96)
  );
}

.video-card__play {
  position: absolute;
  left: 18px;
  bottom: 44px;
  width: 0;
  height: 0;
  border-top: 7px solid transparent;
  border-bottom: 7px solid transparent;
  border-left: 10px solid rgba(255, 255, 255, 0.55);
}

.video-card__line {
  position: absolute;
  left: 18px;
  right: 18px;
  bottom: 20px;
  height: 4px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.38);
}

.video-card__dots,
.video-card__dots::before,
.video-card__dots::after {
  position: absolute;
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.4);
  content: "";
}

.video-card__dots {
  right: 18px;
  bottom: 44px;
}

.video-card__dots::before {
  top: -8px;
  left: 0;
}

.video-card__dots::after {
  top: 8px;
  left: 0;
}

.video-card__title {
  color: #ffffff;
  font-size: 14px;
  line-height: 1.2;
  text-align: center;
  white-space: nowrap;
}
</style>
