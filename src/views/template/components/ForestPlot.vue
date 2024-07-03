<template>
  <div ref="chart"></div>
</template>

<script lang="ts" setup>
import * as d3 from "d3";
import { ref } from "vue";

const chart = ref(null as any);
const data = ref([] as any);

const drawChart = () => {
  // 定义图表的宽度和高度
  const margin = { top: 20, right: 30, bottom: 40, left: 200 },
    width: any = 960 - margin.left - margin.right,
    height: any = data.value.length * 30 + margin.top + margin.bottom;

  // 清除以前的图表（如果有）
  d3.select(chart.value).select("svg").remove();

  // 创建新的 SVG 元素
  const svg: any = d3
    .select(chart.value)
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  // 定义 x 轴和 y 轴的比例尺
  const x: any = d3.scaleLinear().domain([0, 2]).range([0, width]);
  const y: any = d3
    .scaleBand()
    .domain(data.value.map((d: any) => d.label))
    .range([0, height - margin.top - margin.bottom])
    .padding(0.1);

  // 添加 x 轴，并将其移到底部
  svg
    .append("g")
    .attr("transform", `translate(0,${height - margin.top - margin.bottom})`)
    .call(d3.axisBottom(x));

  // 添加 y 轴
  svg.append("g").call(d3.axisLeft(y));

  // 绘制置信区间的线
  svg
    .selectAll(".ci-line")
    .data(data.value)
    .enter()
    .append("line")
    .attr("class", "ci-line")
    .attr("x1", (d: any) => x(d.ci[0]))
    .attr("x2", (d: any) => x(d.ci[1]))
    .attr("y1", (d: any) => y(d.label) + y.bandwidth() / 2)
    .attr("y2", (d: any) => y(d.label) + y.bandwidth() / 2)
    .attr("stroke", "black");

  // 绘制点估计
  svg
    .selectAll(".point")
    .data(data.value)
    .enter()
    .append("circle")
    .attr("class", "point")
    .attr("cx", (d: any) => x(d.pointEstimate))
    .attr("cy", (d: any) => y(d.label) + y.bandwidth() / 2)
    .attr("r", 5)
    .attr("fill", "red");

  // 添加 p 值标签
  svg
    .selectAll(".p-value")
    .data(data.value)
    .enter()
    .append("text")
    .attr("class", "p-value")
    .attr("x", (d: any) => x(d.ci[1]) + 5)
    .attr("y", (d: any) => y(d.label) + y.bandwidth() / 2)
    .attr("dy", ".35em")
    .text((d: any) => `p=${d.p}`);
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
