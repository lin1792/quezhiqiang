<template>
  <div ref="chart"></div>
</template>

<script lang="ts" setup>
import * as d3 from "d3";
import { ref } from "vue";
interface DataSeries {
  name: string;
  pointEstimate: number;
  ci: [number, number];
}

interface DataItem {
  label: string;
  series: DataSeries[];
}
const chart = ref(null as any);
const data = ref<DataItem[]>([]);

const drawChart = () => {
  const margin = { top: 20, right: 30, bottom: 40, left: 200 },
    width = 960 - margin.left - margin.right,
    height = data.value.length * 50 + margin.top + margin.bottom;

  d3.select(chart.value).select("svg").remove();

  const svg = d3
    .select(chart.value)
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  const x = d3.scaleLinear().domain([0, 2]).range([0, width]);
  const y = d3
    .scaleBand()
    .domain(data.value.map((d: any) => d.id))
    .range([0, height - margin.top - margin.bottom])
    .padding(0.5);

  svg
    .append("g")
    .attr("transform", `translate(0,${height - margin.top - margin.bottom})`)
    .call(d3.axisBottom(x));

  // 创建 y 轴，并将其位置移动到 x 轴的 1.0 位置
  svg
    .append("g")
    .attr("transform", `translate(${x(1.0)},0)`)
    .call(d3.axisLeft(y).tickSize(0).tickFormat("")); // 将 Y 轴移动到 X 轴的 1.0 位置

  const color = d3.scaleOrdinal(d3.schemeCategory10);

  data.value.forEach((d: DataItem) => {
    d.series.forEach((s: DataSeries, i: number) => {
      const yPos = y(d.id)! + ((i + 1) * y.bandwidth()) / (d.series.length + 1);

      // 绘制置信区间的线
      svg
        .append("line")
        .attr("class", "ci-line")
        .attr("x1", x(s.ci[0]))
        .attr("x2", x(s.ci[1]))
        .attr("y1", yPos)
        .attr("y2", yPos)
        .attr("stroke", color(i.toString()));

      // 绘制点估计
      svg
        .append("circle")
        .attr("class", "point")
        .attr("cx", x(s.pointEstimate))
        .attr("cy", yPos)
        .attr("r", s.pointEstimate ? 5 : 0)
        .attr("fill", color(i.toString()));

      // 添加系列标签
      svg
        .append("text")
        .attr("class", "series-label")
        .attr("x", x(s.ci[1]) + 5)
        .attr("y", yPos)
        .attr("dy", ".35em")
        .attr("fill", color(i.toString()))
        .text(s.name);

      // 添加置信区间两个端点的圆点
      svg
        .append("circle")
        .attr("class", "ci-point")
        .attr("cx", x(s.ci[0]))
        .attr("cy", yPos)
        .attr("r", s.ci[0] ? 2 : 0)
        .attr("fill", color(i.toString()));

      svg
        .append("circle")
        .attr("class", "ci-point")
        .attr("cx", x(s.ci[1]))
        .attr("cy", yPos)
        .attr("r", s.ci[1] ? 2 : 0)
        .attr("fill", color(i.toString()));

      // 添加自定义的 y 轴标签
      svg
        .append("text")
        .attr("class", "y-axis-label")
        .attr("x", -10)
        .attr("y", yPos)
        .attr("dy", ".35em")
        .attr("text-anchor", "end")
        .attr("fill", "black")
        .text(d.label);
    });
  });
};
defineExpose({
  drawChart,
  data
});
</script>

<style lang="scss" scoped>
svg {
  font: 10px sans-serif;
}
.axis path,
.axis line {
  fill: none;
  shape-rendering: crispEdges;
}
.ci-line {
  stroke: black;
  stroke-width: 1px;
}
.point {
  fill: red;
  stroke: black;
  stroke-width: 1px;
}
.p-value {
  font-size: 12px;
  fill: black;
}
</style>
