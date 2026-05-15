<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from "vue";
import { BarChart, LineChart, PieChart } from "echarts/charts";
import {
  GraphicComponent,
  GridComponent,
  TooltipComponent,
  type GraphicComponentOption,
  type GridComponentOption,
  type TooltipComponentOption,
} from "echarts/components";
import {
  init,
  use,
  graphic,
  type ComposeOption,
  type ECharts,
} from "echarts/core";
import type {
  BarSeriesOption,
  LineSeriesOption,
  PieSeriesOption,
} from "echarts/charts";
import { CanvasRenderer } from "echarts/renderers";
import SectionPanel from "../components/SectionPanel.vue";
import overviewIcon from "../assets/png/img/title.png";
import videoDemo from "../assets/png/img/videoDemo.mp4";

use([
  BarChart,
  LineChart,
  PieChart,
  GraphicComponent,
  GridComponent,
  TooltipComponent,
  CanvasRenderer,
]);

type CimChartOption = ComposeOption<
  | GridComponentOption
  | GraphicComponentOption
  | TooltipComponentOption
  | BarSeriesOption
  | LineSeriesOption
  | PieSeriesOption
>;

const populationStats = [
  { label: "业主人数", value: "2318" },
  { label: "访客人数", value: "880" },
  { label: "外来人数", value: "174" },
];

const videoFeeds = [
  { title: "星光广场云台", source: videoDemo },
  { title: "星光广场云台", source: videoDemo },
  { title: "星光广场云台", source: videoDemo },
];

const alarms = [
  { name: "监控1：大西门云台", time: "07:12:18", status: "待派遣" },
  { name: "监控2：A楼南侧", time: "07:13:12", status: "处理中" },
  { name: "监控3：T9北侧", time: "08:12:18", status: "已消警" },
  { name: "监控4：大西门云台", time: "07:12:18", status: "待派遣" },
  { name: "监控5：A楼南侧", time: "07:13:12", status: "处理中" },
  { name: "监控6：T9北侧", time: "08:12:18", status: "已消警" },
  { name: "监控7：大西门云台", time: "07:12:18", status: "待派遣" },
];

const unitSlides = [
  {
    station: "1#冷站（1#主机）",
    startStatus: "启动",
    runStatus: "运行",
    categories: ["蒸发器", "冷凝器"],
    outlet: [-302, -120],
    inlet: [300, 120],
  },
  {
    station: "2#冷站（2#主机）",
    startStatus: "启动",
    runStatus: "运行",
    categories: ["蒸发器", "冷凝器"],
    outlet: [-260, -96],
    inlet: [280, 102],
  },
  {
    station: "3#冷站（3#主机）",
    startStatus: "待机",
    runStatus: "巡检",
    categories: ["蒸发器", "冷凝器"],
    outlet: [-188, -74],
    inlet: [214, 86],
  },
  {
    station: "4#冷站（4#主机）",
    startStatus: "启动",
    runStatus: "运行",
    categories: ["蒸发器", "冷凝器"],
    outlet: [-320, -138],
    inlet: [295, 128],
  },
];

const comfortChartRef = ref<HTMLDivElement | null>(null);
const efficiencyChartRef = ref<HTMLDivElement | null>(null);
const unitChartRef = ref<HTMLDivElement | null>(null);
const currentUnitIndex = ref(0);
let comfortChart: ECharts | null = null;
let efficiencyChart: ECharts | null = null;
let unitChart: ECharts | null = null;
let unitAutoplayTimer: number | undefined;

const efficiencySeries = [18, 24, 39, 34, 66, 55, 41, 33, 9];
const efficiencyLabels = [
  "6-27",
  "6-27",
  "6-27",
  "6-27",
  "6-27",
  "6-27",
  "6-27",
  "6-27",
  "6-27",
];

const statusClassMap: Record<string, string> = {
  待派遣: "pending",
  处理中: "processing",
  已消警: "resolved",
};

const currentUnitSlide = computed(() => unitSlides[currentUnitIndex.value]);

const renderComfortChart = () => {
  if (!comfortChartRef.value) {
    return;
  }

  comfortChart ??= init(comfortChartRef.value);

  const option: CimChartOption = {
    animation: false,
    graphic: [
      {
        type: "text",
        left: "center",
        top: "42%",
        silent: true,
        style: {
          text: "室内温度\n65%",
          fill: "#35e0d6",
          fontSize: 18,
          fontWeight: 700,
          lineHeight: 24,
          align: "center",
          fontFamily: "Microsoft YaHei, PingFang SC, sans-serif",
        },
      },
    ],
    series: [
      {
        type: "pie",
        radius: ["64%", "90%"],
        center: ["50%", "50%"],
        startAngle: 210,
        label: { show: false },
        labelLine: { show: false },
        emphasis: { scale: false },
        data: [
          {
            value: 65,
            itemStyle: {
              color: new graphic.LinearGradient(0, 0, 1, 1, [
                { offset: 0, color: "#3ad7ff" },
                { offset: 1, color: "#2be38f" },
              ]),
            },
          },
          {
            value: 35,
            itemStyle: { color: "rgba(87, 121, 178, 0.28)" },
          },
        ],
      },
    ],
  };

  comfortChart.setOption(option);
};

const renderEfficiencyChart = () => {
  if (!efficiencyChartRef.value) {
    return;
  }

  efficiencyChart ??= init(efficiencyChartRef.value);

  const option: CimChartOption = {
    animation: false,
    grid: {
      top: 14,
      right: 16,
      bottom: 24,
      left: 34,
    },
    tooltip: { trigger: "axis" },
    xAxis: {
      type: "category",
      boundaryGap: false,
      data: efficiencyLabels,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(220, 236, 255, 0.82)",
        fontSize: 11,
      },
      splitLine: { show: false },
    },
    yAxis: {
      type: "value",
      min: 0,
      max: 70,
      interval: 10,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(173, 209, 255, 0.72)",
        fontSize: 11,
      },
      splitLine: {
        lineStyle: {
          color: "rgba(180, 210, 255, 0.24)",
          type: "dashed",
        },
      },
    },
    series: [
      {
        type: "line",
        data: efficiencySeries,
        smooth: true,
        symbol: "circle",
        symbolSize: 6,
        lineStyle: {
          width: 2,
          color: "#d9a8ff",
        },
        itemStyle: {
          color: "#ffffff",
          borderColor: "#ff8bbd",
          borderWidth: 2,
        },
        areaStyle: {
          color: new graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: "rgba(176, 84, 255, 0.42)" },
            { offset: 1, color: "rgba(51, 224, 216, 0.12)" },
          ]),
        },
      },
    ],
  };

  efficiencyChart.setOption(option);
};

const renderUnitChart = () => {
  if (!unitChartRef.value) {
    return;
  }

  unitChart ??= init(unitChartRef.value);
  const currentSlide = currentUnitSlide.value;

  const option: CimChartOption = {
    animation: false,
    grid: {
      top: 16,
      right: 20,
      bottom: 24,
      left: 56,
    },
    tooltip: { trigger: "axis", axisPointer: { type: "shadow" } },
    xAxis: {
      type: "value",
      min: -400,
      max: 300,
      interval: 100,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(188, 215, 255, 0.78)",
        fontSize: 11,
      },
      splitLine: {
        lineStyle: {
          color: "rgba(180, 210, 255, 0.22)",
          type: "dashed",
        },
      },
    },
    yAxis: {
      type: "category",
      data: currentSlide.categories,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(221, 236, 255, 0.88)",
        fontSize: 12,
      },
    },
    series: [
      {
        name: "出水温度",
        type: "bar",
        stack: "temperature",
        data: currentSlide.outlet,
        barWidth: 20,
        itemStyle: {
          color: new graphic.LinearGradient(1, 0, 0, 0, [
            { offset: 0, color: "#f43cff" },
            { offset: 1, color: "#7d22ff" },
          ]),
        },
      },
      {
        name: "进水温度",
        type: "bar",
        stack: "temperature",
        data: currentSlide.inlet,
        barWidth: 20,
        itemStyle: {
          color: new graphic.LinearGradient(0, 0, 1, 0, [
            { offset: 0, color: "#22b7ff" },
            { offset: 1, color: "#47e5d7" },
          ]),
        },
        label: {
          show: true,
          position: "inside",
          color: "#ffffff",
          fontSize: 11,
        },
      },
    ],
  };

  unitChart.setOption(option);
};

const setUnitSlide = (index: number) => {
  currentUnitIndex.value = index;
  renderUnitChart();
};

const advanceUnitSlide = () => {
  setUnitSlide((currentUnitIndex.value + 1) % unitSlides.length);
};

const startUnitAutoplay = () => {
  if (unitAutoplayTimer) {
    window.clearInterval(unitAutoplayTimer);
  }

  unitAutoplayTimer = window.setInterval(() => {
    advanceUnitSlide();
  }, 4000);
};

const handleResize = () => {
  comfortChart?.resize();
  efficiencyChart?.resize();
  unitChart?.resize();
};

onMounted(() => {
  renderComfortChart();
  renderEfficiencyChart();
  renderUnitChart();
  startUnitAutoplay();
  window.addEventListener("resize", handleResize);
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", handleResize);
  if (unitAutoplayTimer) {
    window.clearInterval(unitAutoplayTimer);
  }
  comfortChart?.dispose();
  efficiencyChart?.dispose();
  unitChart?.dispose();
  comfortChart = null;
  efficiencyChart = null;
  unitChart = null;
});
</script>

<template>
  <section class="cim-page">
    <div class="cim-page__side cim-page__side--left">
      <SectionPanel title="安防概况" class="cim-overview-panel">
        <div class="overview-block">
          <div class="overview-block__header">
            <img :src="overviewIcon" alt="" class="overview-block__icon" />
            <div class="overview-block__meta">
              <p>当前社区总人数</p>
              <strong>12530</strong>
            </div>
          </div>

          <div class="overview-block__stats">
            <article
              v-for="item in populationStats"
              :key="item.label"
              class="overview-stat"
            >
              <span>{{ item.label }}</span>
              <strong style="color: #4b91ff">{{ item.value }}</strong>
            </article>
          </div>
        </div>
      </SectionPanel>

      <SectionPanel title="摄像头视频监控" class="cim-video-panel">
        <div class="cim-video-grid">
          <article
            v-for="feed in videoFeeds"
            :key="feed.title"
            class="cim-video-card"
          >
            <div class="cim-video-card__frame">
              <video
                class="cim-video-card__media"
                :src="feed.source"
                autoplay
                loop
                muted
                playsinline
              ></video>
            </div>
            <div class="cim-video-card__title">{{ feed.title }}</div>
          </article>
        </div>
      </SectionPanel>

      <SectionPanel title="报警讯息列表" class="cim-alarm-panel">
        <div class="cim-alarm-list">
          <article
            v-for="item in alarms"
            :key="`${item.name}-${item.time}`"
            class="cim-alarm-item"
          >
            <div class="cim-alarm-item__bar"></div>
            <div class="cim-alarm-item__content">
              <div class="cim-alarm-item__row cim-alarm-item__row--main">
                <span class="cim-alarm-item__name">{{ item.name }}</span>
                <span
                  class="cim-alarm-item__status"
                  :class="statusClassMap[item.status]"
                >
                  {{ item.status }}
                </span>
              </div>
              <div class="cim-alarm-item__time">{{ item.time }}</div>
            </div>
          </article>
        </div>
      </SectionPanel>
    </div>

    <div class="cim-page__side cim-page__side--right">
      <SectionPanel title="舒适度数据统计" class="comfort-panel">
        <div class="comfort-panel__content">
          <div ref="comfortChartRef" class="comfort-panel__chart"></div>
          <div class="comfort-panel__metrics">
            <div class="comfort-panel__metric">
              <div class="comfort-panel__label">室内空气：32%</div>
              <div class="comfort-panel__track">
                <div class="comfort-panel__fill" style="width: 32%"></div>
              </div>
            </div>
            <div class="comfort-panel__metric">
              <div class="comfort-panel__label">室内温度：53.685%</div>
              <div class="comfort-panel__track">
                <div class="comfort-panel__fill" style="width: 53.685%"></div>
              </div>
            </div>
          </div>
        </div>
      </SectionPanel>

      <SectionPanel title="能源数据概况" class="energy-summary-panel">
        <div class="energy-summary">
          <div class="energy-summary__grid">
            <div class="energy-summary__item">
              <span>本月总能耗（单位：kw.h）</span>
              <strong>11787</strong>
            </div>
            <div class="energy-summary__item">
              <span>本年总能耗（单位：kw.h）</span>
              <strong>210688</strong>
            </div>
            <div class="energy-summary__item">
              <span>本日总负荷（单位：kw.h）</span>
              <strong>73779</strong>
            </div>
            <div class="energy-summary__item">
              <span>本年平均能耗（单位：kw.h）</span>
              <strong>10127</strong>
            </div>
            <div class="energy-summary__item">
              <span>本日总电费（单位：元）</span>
              <strong>5646</strong>
            </div>
            <div class="energy-summary__item">
              <span>本年总电费（单位：元）</span>
              <strong>203231</strong>
            </div>
          </div>
        </div>
      </SectionPanel>

      <SectionPanel title="系统能效统计" class="efficiency-panel">
        <div class="efficiency-panel__header">
          <span>位数/时</span>
          <span>单位：千/kw.h</span>
        </div>
        <div ref="efficiencyChartRef" class="efficiency-panel__chart"></div>
      </SectionPanel>

      <SectionPanel title="机组运行情况" class="unit-panel">
        <div class="unit-panel__legend">
          <span><i class="legend-dot legend-dot--out"></i>出水温度</span>
          <span><i class="legend-dot legend-dot--in"></i>进水温度</span>
        </div>
        <div class="unit-panel__content">
          <div class="unit-panel__chart-area">
            <div ref="unitChartRef" class="unit-panel__chart"></div>
            <div class="unit-panel__pager">
              <button
                v-for="(_, index) in unitSlides"
                :key="index"
                type="button"
                :class="{ 'is-active': index === currentUnitIndex }"
                @click="
                  setUnitSlide(index);
                  startUnitAutoplay();
                "
              ></button>
            </div>
          </div>
          <div class="unit-panel__meta">
            <div class="unit-panel__station">
              {{ currentUnitSlide.station }}
            </div>
            <div class="unit-panel__state">
              <span>开启状态</span>
              <strong>{{ currentUnitSlide.startStatus }}</strong>
            </div>
            <div class="unit-panel__state">
              <span>运行状态</span>
              <strong>{{ currentUnitSlide.runStatus }}</strong>
            </div>
          </div>
        </div>
      </SectionPanel>
    </div>
  </section>
</template>

<style scoped>
.cim-page {
  height: 100%;
  min-height: 0;
  display: flex;
  gap: 14px;
}

.cim-page__side {
  display: flex;
  flex-direction: column;
  min-height: 0;
  gap: 14px;
}

.cim-page__side--left {
  width: 450px;
  max-width: 450px;
}

.cim-page__side--right {
  width: 420px;
  max-width: 420px;
  margin-left: auto;
}

.cim-overview-panel {
  height: 22%;
}

.cim-video-panel {
  height: 17%;
}

.cim-alarm-panel {
  flex: 1;
  min-height: 0;
}

.comfort-panel {
  height: 20%;
}

.energy-summary-panel {
  height: 28%;
}

.efficiency-panel {
  height: 22%;
}

.unit-panel {
  /* flex: 1; */
  height: 30%;
  /* background-color: #17d3a8; */
  min-height: 0;
}

:deep(.cim-overview-panel .section-panel__header),
:deep(.cim-video-panel .section-panel__header),
:deep(.cim-alarm-panel .section-panel__header),
:deep(.comfort-panel .section-panel__header),
:deep(.energy-summary-panel .section-panel__header),
:deep(.efficiency-panel .section-panel__header),
:deep(.unit-panel .section-panel__header) {
  padding-bottom: 6px;
}

:deep(.cim-overview-panel .section-panel__body),
:deep(.cim-video-panel .section-panel__body),
:deep(.comfort-panel .section-panel__body),
:deep(.energy-summary-panel .section-panel__body),
:deep(.efficiency-panel .section-panel__body),
:deep(.unit-panel .section-panel__body) {
  padding-top: 0;
}

.overview-block {
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.overview-block__header {
  display: grid;
  grid-template-columns: 74px minmax(0, 1fr);
  align-items: center;
  column-gap: 14px;
}

.overview-block__icon {
  width: 54px;
  height: 54px;
  object-fit: contain;
  justify-self: center;
}

.overview-block__meta p {
  margin: 0 0 6px;
  color: #84b6ff;
  font-size: 13px;
}

.overview-block__meta strong {
  color: #5ed0ff;
  font-size: 34px;
  font-weight: 500;
  line-height: 1;
}

.overview-block__stats {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px;
}

.overview-stat {
  text-align: center;
}

.overview-stat span {
  display: block;
  padding: 4px 0;
  margin-bottom: 6px;
  color: #ffffff;
  font-size: 12px;
  background: linear-gradient(
    90deg,
    rgba(22, 147, 217, 0.96),
    rgba(110, 84, 220, 0.96)
  );
}

.overview-stat strong {
  color: #edf5ff;
  font-size: 17px;
  font-weight: 500;
}

.cim-video-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 10px;
}

.cim-video-card {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.cim-video-card__frame {
  position: relative;
  aspect-ratio: 16 / 9;
  overflow: hidden;
  background: rgba(6, 10, 22, 0.76);
  box-shadow: inset 0 0 0 1px rgba(255, 255, 255, 0.04);
}

.cim-video-card__media {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.cim-video-card__title {
  color: #ffffff;
  font-size: 12px;
  text-align: center;
  white-space: nowrap;
}

.cim-alarm-list {
  display: flex;
  flex-direction: column;
  gap: 6px;
  min-height: 0;
  overflow: hidden;
}

.cim-alarm-item {
  display: grid;
  grid-template-columns: 16px minmax(0, 1fr);
  align-items: stretch;
  min-height: 64px;
}

.cim-alarm-item__bar {
  background: linear-gradient(
    180deg,
    rgba(41, 145, 255, 0.9),
    rgba(22, 95, 202, 0.92)
  );
}

.cim-alarm-item__content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 10px;
  padding: 0 0 0 16px;
  background: linear-gradient(
    90deg,
    rgba(2, 10, 28, 0.52),
    rgba(2, 10, 28, 0.12)
  );
}

.cim-alarm-item__row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.cim-alarm-item__name,
.cim-alarm-item__time {
  color: #ffffff;
  font-size: 15px;
}

.cim-alarm-item__status {
  flex-shrink: 0;
  min-width: 50px;
  padding: 3px 8px;
  border-radius: 4px;
  color: #ffffff;
  font-size: 12px;
  text-align: center;
}

.cim-alarm-item__status.pending {
  background: #7441d7;
}

.cim-alarm-item__status.processing {
  background: #489cff;
}

.cim-alarm-item__status.resolved {
  background: #355dd8;
}

.comfort-panel__content {
  display: grid;
  grid-template-columns: 160px minmax(0, 1fr);
  align-items: center;
  gap: 16px;
}

.comfort-panel__chart {
  width: 138px;
  height: 138px;
  margin: 0 auto;
}

.comfort-panel__metrics {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.comfort-panel__label {
  margin-bottom: 6px;
  color: #ffffff;
  font-size: 14px;
}

.comfort-panel__track {
  height: 16px;
  background: transparent;
}

.comfort-panel__fill {
  height: 100%;
  background: linear-gradient(90deg, #1aa6d4, #1bd3a9);
}

.energy-summary {
  height: 100%;
}

.energy-summary__grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 14px 18px;
}

.energy-summary__item {
  text-align: center;
}

.energy-summary__item span {
  display: block;
  margin-bottom: 4px;
  color: #ffffff;
  font-size: 13px;
  line-height: 1.3;
}

.energy-summary__item strong {
  color: #48cfff;
  font-size: 28px;
  font-weight: 500;
}

.efficiency-panel__header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 4px;
  color: #7aa7ff;
  font-size: 13px;
}

.efficiency-panel__chart {
  width: 100%;
  flex: 1;
  min-height: 0;
}

.unit-panel__legend {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 8px;
  color: #ffffff;
  font-size: 13px;
}

.legend-dot {
  display: inline-block;
  width: 22px;
  height: 14px;
  margin-right: 6px;
  border-radius: 3px;
  vertical-align: middle;
}

.legend-dot--out {
  background: linear-gradient(90deg, #7d22ff, #f43cff);
}

.legend-dot--in {
  background: linear-gradient(90deg, #22b7ff, #47e5d7);
}

.unit-panel__content {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 92px;
  gap: 12px;
  min-height: 0;
  flex: 1;
}

.unit-panel__chart-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 0;
  min-height: 0;
}

.unit-panel__chart {
  width: 100%;
  min-height: 0;
  flex: 1;
}

.unit-panel__meta {
  display: flex;
  flex-direction: column;
  gap: 14px;
  align-items: stretch;
}

.unit-panel__station {
  padding: 4px 8px;
  background: rgba(77, 90, 182, 0.9);
  color: #ffffff;
  font-size: 14px;
  text-align: center;
  line-height: 1.4;
}

.unit-panel__state {
  display: flex;
  flex-direction: column;
  gap: 6px;
  color: #ffffff;
  font-size: 13px;
}

.unit-panel__state strong {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-height: 26px;
  background: linear-gradient(180deg, #19b6ff, #17d3a8);
  color: #ffffff;
  font-size: 16px;
  font-weight: 600;
}

.unit-panel__pager {
  display: flex;
  justify-content: center;
  gap: 18px;
  width: 100%;
  padding-top: 8px;
}

.unit-panel__pager button {
  width: 8px;
  height: 8px;
  padding: 0;
  border: 0;
  border-radius: 50%;
  background: rgba(115, 166, 255, 0.56);
  cursor: pointer;
}

.unit-panel__pager button.is-active {
  background: #4ca4ff;
  box-shadow: 0 0 10px rgba(76, 164, 255, 0.7);
}
</style>
