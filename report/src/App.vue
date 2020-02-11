<template>
	<div id="wrapper">
		abcd
	</div>
</template>

<script>
import * as d3 from "d3";

export default {
	name: "App",
	mounted() {
		async function drawLineChart() {
			const dataset = await d3.json("/my_weather_data.json");
			// dateParser 可以解析，以 年-月-日 为格式的日期字符串
			const dateParser = d3.timeParse("%Y-%m-%d");
			const xAccessor = d => dateParser(d.date);
			const yAccessor = d => d.temperatureMax;
			console.log(yAccessor);
			console.log(xAccessor(dataset[0]) instanceof Date);
		}
		drawLineChart();
	}
};
</script>

<style>
body {
	margin: 0;
}
</style>
