<template>
	<div ref="wrapper">
		abcd
	</div>
</template>

<script>
import * as d3 from "d3";

export default {
	name: "App",
	computed: {
		wrapper() {
			return d3.select(this.$refs.wrapper);
		}
	},
	mounted() {
		const vm = this;
		async function drawLineChart() {
			const dataset = await d3.json("/my_weather_data.json");
			// dateParser 可以解析，以 年-月-日 为格式的日期字符串
			const dateParser = d3.timeParse("%Y-%m-%d");
			const xAccessor = d => dateParser(d.date);
			const yAccessor = d => d.temperatureMax;

			// wrapper 的 dimensions
			let dimensions = {
				width: window.innerWidth * 0.9,
				height: 400,
				margin: {
					top: 15,
					right: 15,
					bottom: 40,
					left: 60
				}
			};
			// 计算 bounds 的尺寸
			dimensions.boundedWidth = dimensions.width - dimensions.margin.left - dimensions.margin.right;
			dimensions.boundedHeight = dimensions.height - dimensions.margin.top - dimensions.margin.bottom;
			console.log(vm.wrapper);
			// 添加新的 SVG 元素
			const svgTag = vm.wrapper
				.append("svg")
				.attr("width", dimensions.width)
				.attr("height", dimensions.height);

			// 给 svgTag append g 标签，用来组织内部的图表
			const bounds = svgTag
				.append("g")
				.style("transform", `translate(${dimensions.margin.left}px, ${dimensions.margin.top}px)`);
			console.log(bounds);

			// 创建一个 温度 → y轴 的线性缩放
			const yScale = d3
				.scaleLinear()
				.domain(d3.extent(dataset, yAccessor))
				.range([dimensions.boundedHeight, 0]);

			// 添加一个矩形，来盖住所有低于 freezing 的温度
			const freezingTemperaturePlacement = yScale(32);
			const freezingTemperatures = bounds
				.append("rect")
				.attr("ry", 10)
				.attr("x", 0)
				.attr("width", dimensions.boundedWidth)
				.attr("y", freezingTemperaturePlacement)
				.attr("height", dimensions.boundedHeight - freezingTemperaturePlacement)
				.attr("fill", "#e0f3f3");

			// 设置 x 轴的 scale ，用 scaleTime 来处理时间
			const xScale = d3
				.scaleTime()
				.domain(d3.extent(dataset, xAccessor))
				.range([0, dimensions.boundedWidth]);

			// d3-shape 的 d3.line() 可以用来生成 path 的 d
			const lineGenerator = d3
				.line()
				.x(d => xScale(xAccessor(d)))
				.y(d => yScale(yAccessor(d)));

			// 给 bounds 加上一个没有 d 的path
			const line = bounds
				.append("path")
				// 而这个 path 的 d 就交给 lineGenerator 了
				.attr("d", lineGenerator(dataset))
				.attr("fill", "none")
				.attr("stroke", "#af9358")
				.attr("stroke-width", 2);
			console.log(freezingTemperatures, line);

			// 创建 label 在 y 轴左侧的轴
			const yAxisGenerator = d3.axisLeft().scale(yScale);
			// axis generator 会创建很多元素，让我们把它们都放进一个 g 中
			const yAxis = bounds.append("g").call(yAxisGenerator);
			console.log(yAxis);
			// 创建 x 轴
			const xAxisGenerator = d3.axisBottom().scale(xScale);
			// 建个 g
			const xAxis = bounds
				.append("g")
				.call(xAxisGenerator)
				.style("transform", `translateY(${dimensions.boundedHeight}px)`);
			console.log(xAxis);
		}
		drawLineChart();
	}
};
</script>

<style>
svg {
	display: block;
}
body {
	margin: 0;
}
</style>
