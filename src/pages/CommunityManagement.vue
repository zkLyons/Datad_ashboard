<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from "vue";
import { BarChart, LineChart, PieChart } from "echarts/charts";
import {
  GraphicComponent,
  GridComponent,
  LegendComponent,
  TooltipComponent,
  type GraphicComponentOption,
  type GridComponentOption,
  type LegendComponentOption,
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
import titleIcon from "../assets/png/img/title.png";
import userIcon from "../assets/png/img/user.png";
import yzIcon from "../assets/png/img/yz.png";

use([
  BarChart,
  LineChart,
  PieChart,
  GraphicComponent,
  GridComponent,
  LegendComponent,
  TooltipComponent,
  CanvasRenderer,
]);

type DashboardChartOption = ComposeOption<
  | GridComponentOption
  | LegendComponentOption
  | GraphicComponentOption
  | TooltipComponentOption
  | LineSeriesOption
  | BarSeriesOption
  | PieSeriesOption
>;

const overviewStats = [
  { label: "归主人数", value: "2318" },
  { label: "访客人数", value: "880" },
  { label: "外来人数", value: "174" },
];

const alarms = [
  { name: "监控1：大西门云台", time: "07:12:18", status: "待派遣" },
  { name: "监控2：A楼南侧", time: "07:13:12", status: "处理中" },
  { name: "监控3：T9北侧", time: "08:12:18", status: "已消警" },
  { name: "监控4：大西门云台", time: "07:12:18", status: "待派遣" },
  { name: "监控5：A楼南侧", time: "07:13:12", status: "处理中" },
  { name: "监控6：T9北侧", time: "08:12:18", status: "已消警" },
  { name: "监控7：大西门云台", time: "07:12:18", status: "待派遣" },
  { name: "监控8：A楼南侧", time: "07:13:12", status: "处理中" },
  { name: "监控9：T9北侧", time: "08:12:18", status: "已消警" },
];

const visitorBars = [
  { label: "6-27", value: 0, color: "#3959aa" },
  { label: "6-28", value: 0, color: "#39d5cd" },
  { label: "6-29", value: 160, color: "#628ff1" },
  { label: "6-27", value: 290, color: "#39d5cd" },
  { label: "6-27", value: 345, color: "#628ff1" },
  { label: "6-27", value: 285, color: "#39d5cd" },
  { label: "6-27", value: 180, color: "#628ff1" },
];

const careSegments = [
  { label: "长期空置", value: 18, color: "#5e62ff" },
  { label: "长期未外出", value: 26, color: "#45e0d9" },
  { label: "小孩独自出门超时", value: 31, color: "#a45bee" },
  { label: "老人独自出门超时", value: 25, color: "#9afb56" },
];

const alertPoints = [20, 28, 40, 34, 66, 54, 42, 33, 12];
const alertLabels = [
  "6-19",
  "6-20",
  "6-21",
  "6-22",
  "6-23",
  "6-24",
  "6-25",
  "6-26",
  "6-27",
];
const visitorChartRef = ref<HTMLDivElement | null>(null);
const careChartRef = ref<HTMLDivElement | null>(null);
const alertChartRef = ref<HTMLDivElement | null>(null);
let visitorChart: ECharts | null = null;
let careChart: ECharts | null = null;
let alertChart: ECharts | null = null;

const statusClassMap: Record<string, string> = {
  待派遣: "pending",
  处理中: "processing",
  已消警: "resolved",
};
const renderVisitorChart = () => {
  if (!visitorChartRef.value) {
    return;
  }

  visitorChart ??= init(visitorChartRef.value);
  const option: DashboardChartOption = {
    animation: false,
    grid: {
      top: 18,
      right: 6,
      bottom: 22,
      left: 30,
      containLabel: false,
    },
    tooltip: { trigger: "axis" },
    xAxis: {
      type: "category",
      data: visitorBars.map((item) => item.label),
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(219, 234, 255, 0.8)",
        fontSize: 12,
      },
    },
    yAxis: {
      type: "value",
      min: 0,
      max: 400,
      interval: 100,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(168, 202, 255, 0.72)",
        fontSize: 12,
      },
      splitLine: {
        lineStyle: {
          color: "rgba(185, 212, 255, 0.38)",
          type: "dashed",
        },
      },
    },
    series: [
      {
        type: "bar",
        data: visitorBars.map((item) => ({
          value: item.value,
          itemStyle: { color: item.color },
        })),
        barWidth: "54%",
      },
    ],
  };

  visitorChart.setOption(option);
};

const renderCareChart = () => {
  if (!careChartRef.value) {
    return;
  }

  careChart ??= init(careChartRef.value);
  const option: DashboardChartOption = {
    animation: false,
    tooltip: { trigger: "item" },
    graphic: [
      {
        type: "text",
        left: "center",
        top: "36%",
        silent: true,
        style: {
          text: "业主关怀",
          fill: "#4faeff",
          fontSize: 13,
          fontFamily: "Microsoft YaHei, PingFang SC, sans-serif",
          fontWeight: 500,
          align: "center",
        },
      },
      {
        type: "image",
        left: "center",
        top: "52%",
        silent: true,
        style: {
          image: yzIcon,
          width: 24,
          height: 24,
        },
      },
    ],
    series: [
      {
        type: "pie",
        radius: ["62%", "92%"],
        center: ["50%", "50%"],
        avoidLabelOverlap: false,
        label: { show: false },
        labelLine: { show: false },
        itemStyle: {
          borderWidth: 0,
        },
        emphasis: {
          scale: false,
        },
        data: careSegments.map((item) => ({
          value: item.value,
          name: item.label,
          itemStyle: { color: item.color },
        })),
      },
    ],
  };

  careChart.setOption(option);
};

const renderAlertChart = () => {
  if (!alertChartRef.value) {
    return;
  }

  alertChart ??= init(alertChartRef.value);
  const option: DashboardChartOption = {
    animation: false,
    grid: {
      top: 18,
      right: 10,
      bottom: 28,
      left: 36,
      containLabel: false,
    },
    xAxis: {
      type: "category",
      boundaryGap: false,
      data: alertLabels,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(219, 234, 255, 0.82)",
        fontSize: 11,
        margin: 10,
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
        color: "rgba(168, 202, 255, 0.72)",
        fontSize: 12,
        margin: 16,
      },
      splitLine: {
        show: true,
        lineStyle: {
          color: "rgba(185, 212, 255, 0.34)",
          type: "dashed",
        },
      },
    },
    series: [
      {
        data: alertPoints,
        type: "line",
        smooth: false,
        symbol: "circle",
        symbolSize: 6,
        lineStyle: {
          width: 2,
          color: "#ff9ad8",
        },
        itemStyle: {
          color: "#ffffff",
          borderColor: "#ff9ad8",
          borderWidth: 1.5,
        },
        areaStyle: {
          color: new graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: "rgba(188, 92, 255, 0.65)" },
            { offset: 1, color: "rgba(46, 160, 255, 0.15)" },
          ]),
        },
      },
    ],
    tooltip: { trigger: "axis" },
  };

  alertChart.setOption(option);
};

const handleChartResize = () => {
  visitorChart?.resize();
  careChart?.resize();
  alertChart?.resize();
};

onMounted(() => {
  renderVisitorChart();
  renderCareChart();
  renderAlertChart();
  window.addEventListener("resize", handleChartResize);
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", handleChartResize);
  visitorChart?.dispose();
  visitorChart = null;
  careChart?.dispose();
  careChart = null;
  alertChart?.dispose();
  alertChart = null;
});
</script>

<template>
  <section class="community-page">
    <div class="community-page__side community-page__side--left">
      <SectionPanel title="安防概况" class="overview-panel">
        <div class="overview">
          <div class="overview__header">
            <img :src="titleIcon" alt="" class="overview__icon" />
            <div class="overview__stats">
              <div class="overview__total">
                <p class="overview__label">当前社区总人数</p>
                <strong>12530</strong>
              </div>
              <div class="overview__blacklist">
                <p class="overview__label">黑名单</p>
                <strong>25</strong>
              </div>
            </div>
          </div>

          <div class="overview__cards">
            <article
              v-for="item in overviewStats"
              :key="item.label"
              class="overview__card"
            >
              <span>{{ item.label }}</span>
              <strong>{{ item.value }}</strong>
            </article>
          </div>
        </div>
      </SectionPanel>

      <SectionPanel title="报警信息列表" class="alarm-panel">
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

    <div class="community-page__spacer" aria-hidden="true"></div>

    <div class="community-page__side community-page__side--right">
      <SectionPanel title="黑名单数据" class="black_manaer">
        <div class="metric-card">
          <img :src="userIcon" alt="" class="metric-card__icon" />
          <div class="metric-card__content">
            <p class="metric-card__label">近一月出现黑名单次数</p>
            <strong>754</strong>
          </div>
        </div>
      </SectionPanel>

      <SectionPanel title="超时访客数据">
        <div class="bar-chart">
          <div class="bar-chart__header">近一周访客超时楼栋分布</div>
          <div ref="visitorChartRef" class="bar-chart__chart"></div>
        </div>
      </SectionPanel>

      <SectionPanel title="业主关怀" class="guanhuai">
        <div class="care-panel">
          <div class="care-panel__chart-wrap">
            <div ref="careChartRef" class="care-panel__chart"></div>
          </div>
          <div class="care-panel__legend">
            <div
              v-for="item in careSegments"
              :key="item.label"
              class="care-panel__legend-item"
            >
              <span
                class="care-panel__swatch"
                :style="{ background: item.color }"
              ></span>
              <span>{{ item.label }}</span>
            </div>
          </div>
        </div>
      </SectionPanel>

      <SectionPanel title="报警数据" class="baojing">
        <div class="line-chart">
          <div class="line-chart__header">
            <span>近一月报警次数</span>
            <span>单位：次数/天</span>
          </div>
          <div ref="alertChartRef" class="line-chart__chart"></div>
        </div>
      </SectionPanel>
    </div>
  </section>
</template>

<style scoped>
.community-page {
  height: 100%;
  min-height: 0;
  display: grid;
  grid-template-columns: minmax(250px, 19.5%) minmax(0, 1fr) minmax(280px, 23%);
  gap: 14px;
}

.community-page__side {
  display: flex;
  flex-direction: column;
  min-height: 0;
  gap: 14px;
}

.community-page__side--right {
  display: flex;
  flex-direction: column;
  min-height: 0;
  max-width: 360px;
  justify-self: end;
}

.community-page__spacer {
  min-width: 0;
}
.overview-panel {
  height: 20%;
}

:deep(.overview-panel .section-panel__header) {
  padding-bottom: 6px;
}

:deep(.overview-panel .section-panel__body) {
  padding-top: 0;
  padding-bottom: 14px;
}

.overview__header {
  display: contents;
}

.overview {
  display: grid;
  grid-template-columns: 116px minmax(0, 1fr) 116px;
  row-gap: 12px;
  column-gap: 12px;
  align-items: start;
}

.overview__icon {
  grid-column: 1;
  grid-row: 1;
  justify-self: center;
  align-self: center;
  width: 52px;
  height: 52px;
  object-fit: contain;
}

.overview__stats {
  grid-column: 2 / 4;
  grid-row: 1;
  display: grid;
  grid-template-columns: minmax(0, 1fr) 116px;
  column-gap: 12px;
  align-items: start;
  width: auto;
}

.overview__label {
  margin: 0 0 6px;
  color: #79a5ff;
  font-size: 12px;
  line-height: 1;
}

.overview strong {
  color: #e6f0ff;
  font-size: 17px;
  font-weight: 500;
}

.overview__total strong {
  display: block;
  color: #5ec8ff;
  font-size: 26px;
  line-height: 1.05;
}

.overview__total {
  align-self: start;
}

.overview__blacklist {
  min-width: 0;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  padding-top: 4px;
}

.overview__blacklist strong {
  font-size: 18px;
  line-height: 1.05;
}

.overview__cards {
  grid-column: 1 / 4;
  grid-row: 2;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
  margin-top: 12px;
}

.overview__card {
  text-align: center;
  align-self: start;
}

.overview__card span {
  display: block;
  padding: 4px 0;
  margin-bottom: 6px;
  color: #ffffff;
  font-size: 13px;
  line-height: 1.1;
  background: linear-gradient(
    90deg,
    rgba(18, 138, 204, 0.95),
    rgba(109, 85, 213, 0.95)
  );
}

.overview__card strong {
  display: block;
  line-height: 1;
}

.alarm-panel {
  flex: 1;
  min-height: 600px;
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

.metric-card {
  display: flex;
  align-items: center;
  gap: 14px;
  min-height: 100%;
  padding-block: 2px;
}
.black_manaer {
  height: 45%;
}

.metric-card__icon {
  width: 54px;
  height: 54px;
  object-fit: contain;
}

.metric-card__content {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 14px;
  width: 100%;
}

.metric-card__label {
  margin: 0;
  padding: 5px 14px;
  background: #1d1e2c;
  box-shadow: inset 0 0 0 1px rgba(255, 255, 255, 0.03);
  color: #ffffff;
  font-size: 13px;
  line-height: 1.5;
  white-space: nowrap;
}

.metric-card__content strong {
  color: #44d7ff;
  font-size: 34px;
  font-weight: 500;
  line-height: 1;
}

.bar-chart {
  display: flex;
  flex-direction: column;
  min-height: 0;
}

.bar-chart__header {
  margin-bottom: 8px;
  color: #4b91ff;
  font-size: 13px;
}

.bar-chart__chart {
  width: 100%;
  height: 186px;
}
.guanhuai {
  height: 80%;
}
.care-panel {
  display: grid;
  grid-template-columns: 132px minmax(0, 1fr);
  align-items: center;
  gap: 12px;
  min-width: 0;
}

.care-panel__chart-wrap {
  position: relative;
  width: 120px;
  height: 120px;
  margin: 0 auto;
}

.care-panel__chart {
  width: 120px;
  height: 120px;
}

.care-panel__legend {
  display: flex;
  flex-direction: column;
  gap: 10px;
  min-width: 0;
}

.care-panel__legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #ffffff;
  font-size: 13px;
  line-height: 1.1;
  white-space: nowrap;
}

.care-panel__swatch {
  width: 26px;
  height: 14px;
  border-radius: 3px;
  flex-shrink: 0;
}
.baojing {
  height: 120%;
}

.line-chart__header {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 4px;
  color: #ffffff;
  font-size: 12px;
}

.line-chart__chart {
  width: 100%;
  height: 190px;
}

@media (max-width: 1560px) {
  .community-page {
    grid-template-columns: minmax(230px, 19.5%) minmax(0, 1fr) minmax(
        260px,
        22%
      );
  }

  .overview__total strong {
    font-size: 24px;
  }

  .metric-card__content strong {
    font-size: 30px;
  }
}

@media (max-width: 1280px) {
  .community-page {
    grid-template-columns: 1fr;
    height: auto;
    overflow-y: auto;
  }

  .community-page__spacer {
    display: none;
  }
}
</style>
