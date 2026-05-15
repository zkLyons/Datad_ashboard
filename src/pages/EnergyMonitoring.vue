<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from "vue";
import { BarChart, LineChart } from "echarts/charts";
import {
  GridComponent,
  TooltipComponent,
  type GridComponentOption,
  type TooltipComponentOption,
} from "echarts/components";
import {
  graphic,
  init,
  use,
  type ComposeOption,
  type ECharts,
} from "echarts/core";
import { CanvasRenderer } from "echarts/renderers";
import powerIcon from "../assets/png/img/gl.png";
import energyIcon from "../assets/png/img/dl.png";

use([BarChart, LineChart, GridComponent, TooltipComponent, CanvasRenderer]);

type EnergyChartOption = ComposeOption<
  | GridComponentOption
  | TooltipComponentOption
  | import("echarts/charts").BarSeriesOption
  | import("echarts/charts").LineSeriesOption
>;

type PlantCard = {
  id: number;
  title: string;
  power: string;
  energy: string;
  operationMode: string;
  runStatus: string;
  c1Status: string;
  c2Status: string;
};

const plantCards: PlantCard[] = Array.from({ length: 4 }, (_, index) => ({
  id: index + 1,
  title: "4#天棚热泵主机",
  power: "211.8",
  energy: "272322.2",
  operationMode: "制热",
  runStatus: "运行",
  c1Status: "启动",
  c2Status: "启动",
}));

const voltageBars = [
  { label: "1", value: 122 },
  { label: "2", value: 146 },
  { label: "3", value: 42 },
  { label: "4", value: 84 },
  { label: "5", value: 126 },
  { label: "6", value: 146 },
  { label: "7", value: 328 },
];

const currentLabels = [
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
const currentSeries = [22, 30, 40, 35, 66, 55, 43, 34, 12];

const visitorBars = [
  { label: "6-27", value: 8, color: "#5d7de2" },
  { label: "6-28", value: 48, color: "#3ddad0" },
  { label: "6-29", value: 198, color: "#5d86f6" },
  { label: "6-27", value: 330, color: "#3ddad0" },
  { label: "6-27", value: 390, color: "#5d86f6" },
  { label: "6-27", value: 326, color: "#3ddad0" },
  { label: "6-27", value: 220, color: "#6f97ff" },
];

const voltageChartRef = ref<HTMLDivElement | null>(null);
const currentChartRef = ref<HTMLDivElement | null>(null);
const visitorChartRef = ref<HTMLDivElement | null>(null);

let voltageChart: ECharts | null = null;
let currentChart: ECharts | null = null;
let visitorChart: ECharts | null = null;

const renderVoltageChart = () => {
  if (!voltageChartRef.value) {
    return;
  }

  voltageChart ??= init(voltageChartRef.value);

  const option: EnergyChartOption = {
    animation: false,
    grid: {
      top: 18,
      left: 36,
      right: 18,
      bottom: 22,
    },
    tooltip: { trigger: "axis", axisPointer: { type: "shadow" } },
    xAxis: {
      type: "value",
      min: 0,
      max: 350,
      interval: 50,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(184, 211, 255, 0.74)",
        fontSize: 11,
      },
      splitLine: { show: false },
    },
    yAxis: {
      type: "category",
      data: voltageBars.map((item) => item.label),
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(219, 236, 255, 0.82)",
        fontSize: 13,
        margin: 10,
      },
    },
    series: [
      {
        type: "bar",
        data: voltageBars.map((item) => item.value),
        barWidth: 24,
        itemStyle: {
          borderRadius: [0, 2, 2, 0],
          color: new graphic.LinearGradient(0, 0, 1, 0, [
            { offset: 0, color: "#2abcf4" },
            { offset: 1, color: "#b65be5" },
          ]),
        },
      },
    ],
  };

  voltageChart.setOption(option);
};

const renderCurrentChart = () => {
  if (!currentChartRef.value) {
    return;
  }

  currentChart ??= init(currentChartRef.value);

  const option: EnergyChartOption = {
    animation: false,
    grid: {
      top: 16,
      left: 36,
      right: 18,
      bottom: 28,
    },
    tooltip: { trigger: "axis" },
    xAxis: {
      type: "category",
      boundaryGap: false,
      data: currentLabels,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(219, 236, 255, 0.76)",
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
        color: "rgba(175, 205, 252, 0.72)",
        fontSize: 11,
      },
      splitLine: {
        lineStyle: {
          color: "rgba(178, 203, 252, 0.34)",
          type: "dashed",
        },
      },
    },
    series: [
      {
        type: "line",
        data: currentSeries,
        smooth: false,
        symbol: "circle",
        symbolSize: 5,
        lineStyle: {
          width: 2,
          color: "#f49ae6",
        },
        itemStyle: {
          color: "#ffffff",
          borderColor: "#f49ae6",
          borderWidth: 1.5,
        },
        areaStyle: {
          color: new graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: "rgba(187, 82, 255, 0.62)" },
            { offset: 1, color: "rgba(31, 186, 255, 0.14)" },
          ]),
        },
      },
    ],
  };

  currentChart.setOption(option);
};

const renderVisitorChart = () => {
  if (!visitorChartRef.value) {
    return;
  }

  visitorChart ??= init(visitorChartRef.value);

  const option: EnergyChartOption = {
    animation: false,
    grid: {
      top: 16,
      left: 36,
      right: 18,
      bottom: 24,
    },
    tooltip: { trigger: "axis", axisPointer: { type: "shadow" } },
    xAxis: {
      type: "category",
      data: visitorBars.map((item) => item.label),
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(219, 236, 255, 0.76)",
        fontSize: 11,
      },
      splitLine: { show: false },
    },
    yAxis: {
      type: "value",
      min: 0,
      max: 400,
      interval: 100,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(175, 205, 252, 0.72)",
        fontSize: 11,
      },
      splitLine: {
        lineStyle: {
          color: "rgba(178, 203, 252, 0.32)",
          type: "dashed",
        },
      },
    },
    series: [
      {
        type: "bar",
        barWidth: "56%",
        data: visitorBars.map((item) => ({
          value: item.value,
          itemStyle: { color: item.color },
        })),
      },
    ],
  };

  visitorChart.setOption(option);
};

const handleResize = () => {
  voltageChart?.resize();
  currentChart?.resize();
  visitorChart?.resize();
};

onMounted(() => {
  renderVoltageChart();
  renderCurrentChart();
  renderVisitorChart();
  window.addEventListener("resize", handleResize);
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", handleResize);
  voltageChart?.dispose();
  currentChart?.dispose();
  visitorChart?.dispose();
  voltageChart = null;
  currentChart = null;
  visitorChart = null;
});
</script>

<template>
  <section class="energy-monitoring-page">
    <div class="energy-monitoring-page__left">
      <article v-for="item in plantCards" :key="item.id" class="plant-card">
        <header class="plant-card__header">
          <span class="plant-card__bars" aria-hidden="true">
            <i></i>
            <i></i>
            <i></i>
          </span>
          <h3>{{ item.title }}</h3>
        </header>

        <div class="plant-card__metrics">
          <div class="metric-chip">
            <div class="metric-chip__icon metric-chip__icon--power">
              <img :src="powerIcon" alt="" />
            </div>
            <div>
              <span>功率</span>
              <strong>{{ item.power }}</strong>
            </div>
          </div>

          <div class="metric-chip">
            <div class="metric-chip__icon metric-chip__icon--energy">
              <img :src="energyIcon" alt="" />
            </div>
            <div>
              <span>电量</span>
              <strong>{{ item.energy }}</strong>
            </div>
          </div>
        </div>

        <div class="plant-card__status-grid">
          <div class="status-row">
            <span class="status-row__label">机组运行工况</span>
            <span class="status-pill status-pill--amber">{{
              item.operationMode
            }}</span>
          </div>
          <div class="status-row">
            <span class="status-row__label">机组运行状态</span>
            <span class="status-pill status-pill--cyan">{{
              item.runStatus
            }}</span>
          </div>
          <div class="status-row">
            <span class="status-row__label">C1启动状态</span>
            <span class="status-pill status-pill--cyan">{{
              item.c1Status
            }}</span>
          </div>
          <div class="status-row">
            <span class="status-row__label">C2启动状态</span>
            <span class="status-pill status-pill--cyan">{{
              item.c2Status
            }}</span>
          </div>
        </div>
      </article>
    </div>

    <div class="energy-monitoring-page__right">
      <div class="energy-select">
        <select aria-label="选择监测对象">
          <option selected>请选择</option>
        </select>
      </div>

      <section class="chart-card chart-card--voltage">
        <header class="chart-card__header">
          <span class="chart-card__bars" aria-hidden="true">
            <i></i>
            <i></i>
            <i></i>
          </span>
          <h3>三相电压</h3>
        </header>
        <div ref="voltageChartRef" class="chart-card__chart"></div>
      </section>

      <section class="chart-card chart-card--current">
        <header class="chart-card__header">
          <span class="chart-card__bars" aria-hidden="true">
            <i></i>
            <i></i>
            <i></i>
          </span>
          <h3>三相电流</h3>
        </header>
        <div ref="currentChartRef" class="chart-card__chart"></div>
      </section>

      <section class="chart-card chart-card--visitor">
        <header class="chart-card__header">
          <span class="chart-card__bars" aria-hidden="true">
            <i></i>
            <i></i>
            <i></i>
          </span>
          <h3>超时访客数据</h3>
        </header>
        <div ref="visitorChartRef" class="chart-card__chart"></div>
      </section>
    </div>
  </section>
</template>

<style scoped>
.energy-monitoring-page {
  display: grid;
  grid-template-columns: 360px minmax(0, 1fr) 374px;
  gap: 12px;
  height: 100%;
  min-height: 0;
}

.energy-monitoring-page__left,
.energy-monitoring-page__right {
  position: relative;
  display: flex;
  flex-direction: column;
  min-height: 0;
}

.energy-monitoring-page__left {
  grid-column: 1;
  gap: 14px;
  padding-top: 24px;
}

.energy-monitoring-page__right {
  grid-column: 3;
  padding-top: 46px;
}

.plant-card {
  position: relative;
  padding: 0 18px 0 6px;
}

.plant-card::before {
  content: "";
  position: absolute;
  inset: -10px 0;
  background: linear-gradient(
    90deg,
    rgba(8, 20, 44, 0.36),
    rgba(8, 20, 44, 0.12) 68%,
    rgba(8, 20, 44, 0)
  );
  pointer-events: none;
}

.plant-card__header,
.chart-card__header {
  display: flex;
  align-items: center;
  gap: 12px;
}

.plant-card__header h3,
.chart-card__header h3 {
  margin: 0;
  color: #5c89ff;
  font-size: 16px;
  font-weight: 500;
}

.plant-card__bars,
.chart-card__bars {
  display: inline-flex;
  align-items: flex-end;
  gap: 4px;
  width: 22px;
  height: 18px;
}

.plant-card__bars i,
.chart-card__bars i {
  display: block;
  width: 3px;
  border-radius: 999px;
  background: linear-gradient(180deg, #6c7aff, #4f7bff);
}

.plant-card__bars i:nth-child(1),
.chart-card__bars i:nth-child(1) {
  height: 10px;
  opacity: 0.72;
}

.plant-card__bars i:nth-child(2),
.chart-card__bars i:nth-child(2) {
  height: 18px;
}

.plant-card__bars i:nth-child(3),
.chart-card__bars i:nth-child(3) {
  height: 14px;
  opacity: 0.82;
}

.plant-card__metrics {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 14px;
  margin-top: 18px;
}

.metric-chip {
  display: flex;
  align-items: center;
  gap: 12px;
  min-width: 0;
}

.metric-chip__icon {
  display: grid;
  place-items: center;
  flex-shrink: 0;
}

.metric-chip__icon--power {
  width: 54px;
  height: 46px;
}

.metric-chip__icon--energy {
  width: 44px;
  height: 52px;
}

.metric-chip__icon img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.metric-chip span {
  display: block;
  margin-bottom: 4px;
  color: rgba(228, 239, 255, 0.94);
  font-size: 13px;
}

.metric-chip strong {
  display: block;
  color: #ffffff;
  font-size: 24px;
  font-weight: 500;
  line-height: 1;
  letter-spacing: 0.4px;
}

.plant-card__status-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px 16px;
  margin-top: 20px;
}

.status-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
}

.status-row__label {
  color: #ffffff;
  font-size: 15px;
  white-space: nowrap;
}

.status-pill {
  min-width: 40px;
  padding: 4px 10px;
  color: #ffffff;
  font-size: 12px;
  text-align: center;
  white-space: nowrap;
}

.status-pill--amber {
  background: linear-gradient(180deg, #edbb50, #d9982f);
}

.status-pill--cyan {
  background: linear-gradient(180deg, #15e4dc, #0fc6a6);
}

.energy-select {
  width: 100%;
  padding-right: 40px;
  margin-bottom: 28px;
}

.energy-select select {
  width: 100%;
  height: 40px;
  padding: 0 16px;
  border: 1px solid rgba(78, 109, 237, 0.88);
  background: rgba(8, 16, 34, 0.56);
  color: rgba(235, 242, 255, 0.96);
  font-size: 14px;
  outline: none;
}

.chart-card {
  position: relative;
  padding: 0 40px 0 18px;
}

.chart-card::before {
  content: "";
  position: absolute;
  inset: -14px 0 -8px;
  background: linear-gradient(
    180deg,
    rgba(6, 17, 34, 0.34),
    rgba(6, 17, 34, 0.16) 55%,
    rgba(6, 17, 34, 0)
  );
  pointer-events: none;
}

.chart-card--voltage {
  height: 260px;
}

.chart-card--current {
  height: 210px;
  margin-top: 20px;
}

.chart-card--visitor {
  height: 238px;
  margin-top: 22px;
}

.chart-card__chart {
  position: relative;
  width: 100%;
  height: calc(100% - 34px);
  margin-top: 12px;
}

@media (max-width: 1760px) {
  .energy-monitoring-page {
    grid-template-columns: 330px minmax(0, 1fr) 344px;
  }

  .metric-chip strong {
    font-size: 22px;
  }

  .status-row__label {
    font-size: 14px;
  }
}
</style>
