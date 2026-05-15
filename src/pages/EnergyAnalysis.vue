<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from "vue";
import { BarChart, LineChart, PieChart } from "echarts/charts";
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
import type {
  BarSeriesOption,
  LineSeriesOption,
  PieSeriesOption,
} from "echarts/charts";
import { CanvasRenderer } from "echarts/renderers";
import SectionPanel from "../components/SectionPanel.vue";
import analysisIcon from "../assets/png/img/fy.png";

use([
  BarChart,
  LineChart,
  PieChart,
  GridComponent,
  TooltipComponent,
  CanvasRenderer,
]);

type EnergyAnalysisChartOption = ComposeOption<
  | GridComponentOption
  | TooltipComponentOption
  | BarSeriesOption
  | LineSeriesOption
  | PieSeriesOption
>;

type RunningRow = {
  name: string;
  electric: string;
  cost: string;
  load: string;
  average: string;
};

const efficiencyPercent = 65;
const efficiencyEmptyLevels = Array.from({ length: 5 }, () => "");
const efficiencyLevelTag = "一级";

const efficiencyMetrics = [
  { label: "总负荷", value: "32KW", percent: 32 },
  { label: "总能耗", value: "53.685KW", percent: 53.685 },
];

const systemCostBars = [
  { label: "6-27", value: 8, color: "#6488f7" },
  { label: "6-28", value: 48, color: "#42ddd1" },
  { label: "6-29", value: 198, color: "#6488f7" },
  { label: "6-27", value: 332, color: "#42ddd1" },
  { label: "6-27", value: 388, color: "#6488f7" },
  { label: "6-27", value: 326, color: "#42ddd1" },
  { label: "6-27", value: 218, color: "#6488f7" },
];

const costShareSegments = [
  { label: "天棚主机", value: 22, color: "#5e63ff" },
  { label: "天棚水泵", value: 18, color: "#3ed6cb" },
  { label: "新风主机", value: 17, color: "#42dfe0" },
  { label: "新风水泵", value: 16, color: "#6870ff" },
  { label: "地源水泵", value: 14, color: "#28b6f4" },
  { label: "冷却水泵", value: 13, color: "#c412ff" },
];

const energyAnalysisRows = [
  { label: "当前", balance: 132, lowCarbon: 102 },
  { label: "最低", balance: 150, lowCarbon: 96 },
  { label: "最高", balance: 48, lowCarbon: 68 },
  { label: "初始", balance: 88, lowCarbon: 46 },
];

const summaryStats = [
  { label: "本月总能耗（单位：kw.h）", value: "11787" },
  { label: "本年总能耗（单位：kw.h）", value: "210688" },
  { label: "本月总负荷（单位：kw.h）", value: "73779" },
  { label: "本年平均能耗（单位：kw.h）", value: "10127" },
  { label: "本月总电费（单位：元）", value: "5646" },
  { label: "本年总电费（单位：元）", value: "203231" },
];

const efficiencyTrendLabels = [
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
const efficiencyTrendValues = [22, 28, 40, 34, 66, 55, 44, 33, 11];

const runningRows: RunningRow[] = [
  {
    name: "天棚主机",
    electric: "10462.7",
    cost: "5620.3",
    load: "1.8",
    average: "2.3",
  },
  {
    name: "天棚水泵",
    electric: "2345.7",
    cost: "5230.3",
    load: "12.8",
    average: "23.3",
  },
  {
    name: "新风主机",
    electric: "3344.7",
    cost: "5620.3",
    load: "1.8",
    average: "2.3",
  },
  {
    name: "天棚水泵",
    electric: "2354.7",
    cost: "5620.3",
    load: "1.8",
    average: "2.3",
  },
  {
    name: "天棚主机",
    electric: "13455.7",
    cost: "5620.3",
    load: "1.8",
    average: "2.3",
  },
  {
    name: "天棚主机",
    electric: "3422.7",
    cost: "5620.3",
    load: "1.8",
    average: "2.3",
  },
  {
    name: "天棚主机",
    electric: "3422.7",
    cost: "5620.3",
    load: "1.8",
    average: "2.3",
  },
  {
    name: "天棚主机",
    electric: "3422.7",
    cost: "5620.3",
    load: "1.8",
    average: "2.3",
  },
  {
    name: "新风水泵",
    electric: "2864.6",
    cost: "4180.2",
    load: "3.6",
    average: "5.1",
  },
  {
    name: "地源水泵",
    electric: "5188.4",
    cost: "6632.6",
    load: "7.4",
    average: "8.9",
  },
  {
    name: "冷却水泵",
    electric: "1986.2",
    cost: "3021.8",
    load: "2.7",
    average: "4.4",
  },
  {
    name: "新风主机",
    electric: "6123.4",
    cost: "7450.6",
    load: "9.2",
    average: "12.6",
  },
  {
    name: "天棚主机",
    electric: "4521.9",
    cost: "5884.1",
    load: "4.1",
    average: "6.7",
  },
  {
    name: "地源水泵",
    electric: "3765.8",
    cost: "4360.4",
    load: "5.4",
    average: "7.2",
  },
];

const efficiencyChartRef = ref<HTMLDivElement | null>(null);
const systemCostChartRef = ref<HTMLDivElement | null>(null);
const costShareChartRef = ref<HTMLDivElement | null>(null);
const analysisChartRef = ref<HTMLDivElement | null>(null);
const trendChartRef = ref<HTMLDivElement | null>(null);

let efficiencyChart: ECharts | null = null;
let systemCostChart: ECharts | null = null;
let costShareChart: ECharts | null = null;
let analysisChart: ECharts | null = null;
let trendChart: ECharts | null = null;

const renderEfficiencyChart = () => {
  if (!efficiencyChartRef.value) {
    return;
  }

  efficiencyChart ??= init(efficiencyChartRef.value);
  const option: EnergyAnalysisChartOption = {
    animation: false,
    series: [
      {
        type: "pie",
        radius: ["70%", "88%"],
        center: ["50%", "50%"],
        startAngle: 140,
        label: { show: false },
        labelLine: { show: false },
        emphasis: { scale: false },
        data: [
          {
            value: efficiencyPercent,
            itemStyle: {
              color: new graphic.LinearGradient(0, 0, 1, 1, [
                { offset: 0, color: "#24aef6" },
                { offset: 1, color: "#41ecd2" },
              ]),
            },
          },
          {
            value: 100 - efficiencyPercent,
            itemStyle: {
              color: "rgba(84, 128, 172, 0.28)",
            },
          },
        ],
      },
    ],
  };

  efficiencyChart.setOption(option);
};

const renderSystemCostChart = () => {
  if (!systemCostChartRef.value) {
    return;
  }

  systemCostChart ??= init(systemCostChartRef.value);
  const option: EnergyAnalysisChartOption = {
    animation: false,
    grid: {
      top: 12,
      right: 10,
      bottom: 20,
      left: 34,
    },
    tooltip: { trigger: "axis", axisPointer: { type: "shadow" } },
    xAxis: {
      type: "category",
      data: systemCostBars.map((item) => item.label),
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(214, 232, 255, 0.8)",
        fontSize: 11,
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
        color: "rgba(170, 206, 255, 0.7)",
        fontSize: 11,
      },
      splitLine: {
        lineStyle: {
          color: "rgba(182, 214, 255, 0.3)",
          type: "dashed",
        },
      },
    },
    series: [
      {
        type: "bar",
        barWidth: "56%",
        data: systemCostBars.map((item) => ({
          value: item.value,
          itemStyle: { color: item.color },
        })),
      },
    ],
  };

  systemCostChart.setOption(option);
};

const renderCostShareChart = () => {
  if (!costShareChartRef.value) {
    return;
  }

  costShareChart ??= init(costShareChartRef.value);
  const option: EnergyAnalysisChartOption = {
    animation: false,
    tooltip: { trigger: "item" },
    series: [
      {
        type: "pie",
        radius: ["68%", "90%"],
        center: ["50%", "50%"],
        label: { show: false },
        labelLine: { show: false },
        emphasis: { scale: false },
        data: costShareSegments.map((item) => ({
          value: item.value,
          name: item.label,
          itemStyle: { color: item.color },
        })),
      },
    ],
  };

  costShareChart.setOption(option);
};

const renderAnalysisChart = () => {
  if (!analysisChartRef.value) {
    return;
  }

  analysisChart ??= init(analysisChartRef.value);
  const option: EnergyAnalysisChartOption = {
    animation: false,
    grid: {
      top: 16,
      right: 14,
      bottom: 10,
      left: 54,
    },
    tooltip: { trigger: "axis", axisPointer: { type: "shadow" } },
    xAxis: {
      type: "value",
      min: 0,
      max: 180,
      interval: 30,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(179, 208, 249, 0.74)",
        fontSize: 11,
      },
      splitLine: {
        lineStyle: {
          color: "rgba(182, 214, 255, 0.22)",
          type: "dashed",
        },
      },
    },
    yAxis: {
      type: "category",
      data: energyAnalysisRows.map((item) => item.label),
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(222, 236, 255, 0.86)",
        fontSize: 13,
      },
    },
    series: [
      {
        type: "bar",
        name: "地热平衡",
        data: energyAnalysisRows.map((item) => item.balance),
        barWidth: 12,
        itemStyle: {
          color: "#8e4dd7",
        },
      },
      {
        type: "bar",
        name: "节能环保",
        data: energyAnalysisRows.map((item) => item.lowCarbon),
        barWidth: 12,
        itemStyle: {
          color: "#38d7b7",
        },
      },
    ],
  };

  analysisChart.setOption(option);
};

const renderTrendChart = () => {
  if (!trendChartRef.value) {
    return;
  }

  trendChart ??= init(trendChartRef.value);
  const option: EnergyAnalysisChartOption = {
    animation: false,
    grid: {
      top: 18,
      right: 14,
      bottom: 24,
      left: 36,
    },
    tooltip: { trigger: "axis" },
    xAxis: {
      type: "category",
      boundaryGap: false,
      data: efficiencyTrendLabels,
      axisLine: { show: false },
      axisTick: { show: false },
      axisLabel: {
        color: "rgba(219, 236, 255, 0.76)",
        fontSize: 11,
      },
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
        data: efficiencyTrendValues,
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

  trendChart.setOption(option);
};

const handleResize = () => {
  efficiencyChart?.resize();
  systemCostChart?.resize();
  costShareChart?.resize();
  analysisChart?.resize();
  trendChart?.resize();
};

onMounted(() => {
  renderEfficiencyChart();
  renderSystemCostChart();
  renderCostShareChart();
  renderAnalysisChart();
  renderTrendChart();
  window.addEventListener("resize", handleResize);
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", handleResize);
  efficiencyChart?.dispose();
  systemCostChart?.dispose();
  costShareChart?.dispose();
  analysisChart?.dispose();
  trendChart?.dispose();
  efficiencyChart = null;
  systemCostChart = null;
  costShareChart = null;
  analysisChart = null;
  trendChart = null;
});
</script>

<template>
  <section class="energy-analysis-page">
    <div class="energy-analysis-page__side energy-analysis-page__side--left">
      <SectionPanel
        title="当代能效标识"
        class="analysis-panel analysis-panel--efficiency"
      >
        <div class="efficiency-badge">
          <div class="efficiency-badge__levels">
            <span class="efficiency-badge__level-note">低</span>
            <span
              v-for="(_, index) in efficiencyEmptyLevels"
              :key="`empty-${index}`"
              class="efficiency-badge__level"
            ></span>
            <span class="efficiency-badge__level-note">高</span>
            <span class="efficiency-badge__level is-active">
              {{ efficiencyLevelTag }}
            </span>
          </div>

          <div class="efficiency-badge__body">
            <div class="efficiency-badge__chart-wrap">
              <div
                ref="efficiencyChartRef"
                class="efficiency-badge__chart"
              ></div>
              <div class="efficiency-badge__center">
                <span>能效比</span>
                <strong>{{ efficiencyPercent }}%</strong>
              </div>
            </div>

            <div class="efficiency-badge__metrics">
              <div
                v-for="item in efficiencyMetrics"
                :key="item.label"
                class="metric-progress"
              >
                <div class="metric-progress__label">
                  {{ item.label }}：{{ item.value }}
                </div>
                <div class="metric-progress__track">
                  <div
                    class="metric-progress__fill"
                    :style="{ width: `${item.percent}%` }"
                  >
                    <span>{{ item.percent }}%</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </SectionPanel>

      <SectionPanel
        title="系统费用统计"
        class="analysis-panel analysis-panel--cost"
      >
        <div class="panel-unit">元/天</div>
        <div ref="systemCostChartRef" class="analysis-chart"></div>
      </SectionPanel>

      <SectionPanel
        title="各类型费用占比"
        class="analysis-panel analysis-panel--share"
      >
        <div class="cost-share">
          <div class="cost-share__chart-wrap">
            <div ref="costShareChartRef" class="cost-share__chart"></div>
            <div class="cost-share__center">
              <span>费用占比</span>
              <img :src="analysisIcon" alt="" />
            </div>
          </div>

          <div class="cost-share__legend">
            <div
              v-for="item in costShareSegments"
              :key="item.label"
              class="cost-share__legend-item"
            >
              <span
                class="cost-share__legend-swatch"
                :style="{ background: item.color }"
              ></span>
              <span>{{ item.label }}</span>
            </div>
          </div>
        </div>
      </SectionPanel>

      <SectionPanel
        title="能源分析"
        class="analysis-panel analysis-panel--compare"
      >
        <div class="analysis-legend">
          <span
            ><i class="analysis-legend__dot analysis-legend__dot--balance"></i
            >地热平衡</span
          >
          <span
            ><i
              class="analysis-legend__dot analysis-legend__dot--low-carbon"
            ></i
            >节能环保</span
          >
        </div>
        <div ref="analysisChartRef" class="analysis-chart"></div>
      </SectionPanel>
    </div>

    <div class="energy-analysis-page__side energy-analysis-page__side--right">
      <SectionPanel
        title="能源数据概况"
        class="analysis-panel analysis-panel--summary"
      >
        <div class="summary-grid">
          <article
            v-for="item in summaryStats"
            :key="item.label"
            class="summary-grid__item"
          >
            <span>{{ item.label }}</span>
            <strong>{{ item.value }}</strong>
          </article>
        </div>
      </SectionPanel>

      <SectionPanel
        title="系统能效统计"
        class="analysis-panel analysis-panel--trend"
      >
        <div class="panel-unit panel-unit--left" style="margin-left: 30px">
          位数/时
        </div>
        <div class="panel-unit panel-unit--right">单位：千/kw.h</div>
        <div ref="trendChartRef" class="analysis-chart"></div>
      </SectionPanel>

      <SectionPanel
        title="运行相关数据"
        class="analysis-panel analysis-panel--table"
      >
        <div class="data-table">
          <div class="data-table__header">
            <span class="data-table__cell">名称</span>
            <span class="data-table__cell">总电量</span>
            <span class="data-table__cell">总电费</span>
            <span class="data-table__cell">总负荷</span>
            <span class="data-table__cell">平均能耗</span>
          </div>
          <div class="data-table__scroll">
            <div class="data-table__body">
              <div
                v-for="(row, index) in runningRows"
                :key="`${row.name}-${index}`"
                class="data-table__row"
              >
                <span class="data-table__cell">{{ row.name }}</span>
                <span class="data-table__cell">{{ row.electric }}</span>
                <span class="data-table__cell">{{ row.cost }}</span>
                <span class="data-table__cell">{{ row.load }}</span>
                <span class="data-table__cell">{{ row.average }}</span>
              </div>
            </div>
          </div>
        </div>
      </SectionPanel>
    </div>
  </section>
</template>

<style scoped>
.energy-analysis-page {
  display: grid;
  grid-template-columns: 372px minmax(0, 1fr) 430px;
  gap: 12px;
  height: 100%;
  min-height: 0;
}

.energy-analysis-page__side {
  position: relative;
  display: flex;
  flex-direction: column;
  min-height: 0;
  gap: 14px;
}

.energy-analysis-page__side--left {
  grid-column: 1;
  padding-top: 22px;
}

.energy-analysis-page__side--right {
  grid-column: 3;
  padding-top: 22px;
}

.analysis-panel {
  min-height: 0;
  /* background-color: red; */
}

.analysis-panel--efficiency {
  height: 200px;
}

.analysis-panel--cost {
  height: 210px;
}

.analysis-panel--share {
  height: 210px;
}

.analysis-panel--compare {
  flex: 1;
  min-height: 0;
}

.analysis-panel--summary {
  height: 238px;
}

.analysis-panel--trend {
  height: 244px;
}

.analysis-panel--table {
  flex: 1;
  min-height: 0;
  /* background-color: red; */
  /* width: 120%; */
}

.panel-unit {
  margin: -4px 0 0px 4px;
  color: #4f91ff;
  font-size: 14px;
}

.panel-unit--right {
  margin-left: auto;
  margin-right: 2px;
}
.panel-unit--left {
  margin-right: auto;
  margin-left: 2px;
}

.analysis-chart {
  width: 100%;
  flex: 1;
  min-height: 0;
}

.efficiency-badge {
  display: flex;
  flex-direction: column;
  gap: 12px;
  height: 100%;
}

.efficiency-badge__levels {
  display: flex;
  align-items: center;
  gap: 6px;
  color: #ffffff;
  font-size: 13px;
}

.efficiency-badge__level-note {
  color: #ffffff;
}

.efficiency-badge__level {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 16px;
  border: 1px solid rgba(126, 113, 245, 0.78);
  color: transparent;
  font-size: 11px;
}

.efficiency-badge__level.is-active {
  width: 40px;
  border-color: rgba(31, 188, 255, 0.95);
  background: linear-gradient(90deg, #22b7ff, #37d9db);
  color: #ffffff;
}

.efficiency-badge__body {
  display: grid;
  grid-template-columns: 140px minmax(0, 1fr);
  align-items: center;
  gap: 18px;
  min-height: 0;
  flex: 1;
}

.efficiency-badge__chart-wrap {
  position: relative;
  width: 116px;
  height: 116px;
  margin-left: 8px;
}

.efficiency-badge__chart {
  width: 100%;
  height: 100%;
}

.efficiency-badge__center {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  pointer-events: none;
}

.efficiency-badge__center span {
  color: #79d8f9;
  font-size: 15px;
}

.efficiency-badge__center strong {
  color: #ffffff;
  font-size: 38px;
  line-height: 1;
}

.efficiency-badge__metrics {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.metric-progress__label {
  margin-bottom: 8px;
  color: #ffffff;
  font-size: 14px;
}

.metric-progress__track {
  width: 100%;
  height: 18px;
  background: transparent;
}

.metric-progress__fill {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  height: 100%;
  padding-right: 8px;
  background: linear-gradient(90deg, #26bdf3, #3adfd4);
}

.metric-progress__fill span {
  color: #ffffff;
  font-size: 12px;
}

.cost-share {
  display: grid;
  grid-template-columns: 152px minmax(0, 1fr);
  align-items: center;
  gap: 10px;
  height: 100%;
}

.cost-share__chart-wrap {
  position: relative;
  width: 136px;
  height: 136px;
  margin: 0 auto;
}

.costShare__chart,
.cost-share__chart {
  width: 100%;
  height: 100%;
}

.cost-share__center {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 6px;
  pointer-events: none;
}

.cost-share__center span {
  color: #4caeff;
  font-size: 15px;
}

.cost-share__center img {
  width: 30px;
  height: 30px;
  object-fit: contain;
}

.cost-share__legend {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.cost-share__legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #ffffff;
  font-size: 14px;
}

.cost-share__legend-swatch {
  width: 24px;
  height: 14px;
  border-radius: 4px;
}

.analysis-legend {
  display: flex;
  align-items: center;
  gap: 14px;
  margin-bottom: 8px;
  color: #ffffff;
  font-size: 13px;
}

.analysis-legend__dot {
  display: inline-block;
  width: 24px;
  height: 12px;
  margin-right: 6px;
  border-radius: 4px;
  vertical-align: middle;
}

.analysis-legend__dot--balance {
  background: #8e4dd7;
}

.analysis-legend__dot--low-carbon {
  background: #38d7b7;
}

.summary-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 1px 10px;
  /* padding-top: 2px; */
}

.summary-grid__item {
  text-align: center;
}

.summary-grid__item span {
  display: block;
  min-height: 30px;
  color: #ffffff;
  font-size: 13px;
  line-height: 30px;
  /* background-color: #22b7ff; */
}

.summary-grid__item strong {
  display: block;
  /* margin-top: 6px; */
  color: #2ec2ff;
  font-size: 24px;
  font-weight: 500;
  line-height: 1;
}

.data-table {
  display: flex;
  flex-direction: column;
  gap: 5px;
  min-height: 0;
  height: 100%;
  padding-top: 2px;
}

.data-table__header,
.data-table__row {
  display: grid;
  grid-template-columns:
    minmax(0, 1fr) minmax(0, 1fr) minmax(0, 1.12fr)
    minmax(0, 0.8fr) minmax(0, 0.8fr);
  align-items: center;
  column-gap: 5px;
}

.data-table__header {
  padding: 0 12px;
  color: #35c9ff;
  font-size: 14px;
  font-weight: 600;
  letter-spacing: 0.5px;
  text-shadow: 0 0 12px rgba(53, 201, 255, 0.35);
}

.data-table__scroll {
  position: relative;
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  overflow-x: hidden;
  padding-right: 8px;
  padding-bottom: 4px;
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.data-table__scroll::-webkit-scrollbar {
  width: 0;
  height: 0;
}

.data-table__body {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.data-table__row {
  position: relative;
  min-height: 32px;
  padding: 0 12px;
  color: rgba(244, 249, 255, 0.98);
  font-size: 14px;
  font-weight: 600;
  line-height: 32px;
}

.data-table__row::before {
  content: "";
  position: absolute;
  inset: 0;
  opacity: 0;
  background: linear-gradient(
    90deg,
    rgba(48, 122, 220, 0.62),
    rgba(52, 190, 255, 0.24) 54%,
    rgba(39, 103, 198, 0.56)
  );
  box-shadow:
    inset 0 0 0 1px rgba(125, 211, 255, 0.08),
    0 0 16px rgba(32, 129, 214, 0.12);
  pointer-events: none;
}

.data-table__row:nth-child(even)::before {
  opacity: 1;
}

.data-table__cell {
  position: relative;
  z-index: 1;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.data-table__header .data-table__cell:first-child,
.data-table__row .data-table__cell:first-child {
  text-align: left;
}

.data-table__header .data-table__cell:not(:first-child),
.data-table__row .data-table__cell:not(:first-child) {
  text-align: center;
}

@media (max-width: 1760px) {
  .energy-analysis-page {
    grid-template-columns: 340px minmax(0, 1fr) 392px;
  }

  .summary-grid__item strong {
    font-size: 22px;
  }

  .efficiency-badge__center strong {
    font-size: 34px;
  }
}
</style>
