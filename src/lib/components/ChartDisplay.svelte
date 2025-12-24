<script lang="ts">
	import { onDestroy, onMount } from 'svelte';
	import { browser } from '$app/environment';

	export let config: {
		chartType: string;
		xAxis: string;
		yAxis: string;
		yAxes?: string[];
		stacked?: boolean;
		annotations?: { value: number; label: string; color: string }[];
	};
	export let data: any[];

	let chartElement: HTMLElement;
	let chart: any;
	let lastConfigKey = '';
	let lastSeriesKey = '';
	let insights: { title: string; detail: string }[] = [];
	let narrative: string[] = [];
	let dataNotes: string[] = [];

	$: if (chart && config && data) {
		updateChart();
	}

	async function loadApexCharts() {
		if (browser) {
			const module = await import('apexcharts');
			return module.default;
		}
		return null;
	}

	onMount(async () => {
		const ApexCharts = await loadApexCharts();
		if (ApexCharts && chartElement) {
			chart = new ApexCharts(chartElement, getOptions());
			chart.render();
		}
	});

	onDestroy(() => {
		if (chart) {
			chart.destroy();
			chart = null;
		}
	});

	function getConfigKey() {
		return JSON.stringify({
			chartType: config.chartType,
			xAxis: config.xAxis,
			yAxis: config.yAxis,
			yAxes: config.yAxes || [],
			stacked: !!config.stacked,
			annotations: config.annotations || []
		});
	}

	function toNumber(value: any) {
		const num = Number(value);
		return Number.isFinite(num) ? num : null;
	}

	function formatNumber(value: number) {
		return typeof value === 'number' ? value.toLocaleString() : String(value);
	}

	function analyzeData() {
		const rows = Array.isArray(data) ? data : [];
		const activeYAxes = config.yAxes && config.yAxes.length > 0 ? config.yAxes : [config.yAxis];

		if (!rows.length || !activeYAxes[0]) {
			insights = [];
			narrative = [];
			dataNotes = [];
			return;
		}

		const primaryAxis = activeYAxes[0];
		const values = rows
			.map((row) => toNumber(row[primaryAxis]))
			.filter((val) => val !== null) as number[];

		if (values.length === 0) {
			insights = [];
			narrative = [];
			dataNotes = [];
			return;
		}

		const total = values.reduce((sum, val) => sum + val, 0);
		const mean = total / values.length;
		const min = Math.min(...values);
		const max = Math.max(...values);

		const variance =
			values.reduce((sum, val) => sum + Math.pow(val - mean, 2), 0) / values.length;
		const stdDev = Math.sqrt(variance);

		const maxIndex = values.findIndex((val) => val === max);
		const minIndex = values.findIndex((val) => val === min);
		const maxLabel = rows[maxIndex]?.[config.xAxis];
		const minLabel = rows[minIndex]?.[config.xAxis];

		let trendLabel = 'Stable trend';
		if (values.length >= 3) {
			const n = values.length;
			const xMean = (n - 1) / 2;
			const yMean = mean;
			let num = 0;
			let den = 0;
			for (let i = 0; i < n; i += 1) {
				const x = i - xMean;
				num += x * (values[i] - yMean);
				den += x * x;
			}
			const slope = den === 0 ? 0 : num / den;
			const normalized = Math.abs(mean) < 1e-6 ? slope : slope / Math.abs(mean);
			if (normalized > 0.02) trendLabel = 'Upward trend';
			if (normalized < -0.02) trendLabel = 'Downward trend';
		}

		const outliers =
			stdDev === 0
				? 0
				: values.filter((val) => Math.abs((val - mean) / stdDev) > 2.5).length;

		const missingCount = rows.length - values.length;
		const uniqueX = new Set(rows.map((row) => row[config.xAxis])).size;

		insights = [
			{
				title: 'Peak',
				detail: `${formatNumber(max)} at ${maxLabel ?? 'N/A'}`
			},
			{
				title: 'Low',
				detail: `${formatNumber(min)} at ${minLabel ?? 'N/A'}`
			},
			{
				title: 'Trend',
				detail: `${trendLabel} across ${values.length} points`
			}
		];

		if (outliers > 0) {
			insights.push({
				title: 'Outliers',
				detail: `${outliers} values stand out from the average`
			});
		}

		narrative = [
			`${primaryAxis} averages ${formatNumber(mean)} across ${values.length} records.`,
			`The highest point is ${formatNumber(max)} at ${maxLabel ?? 'N/A'}.`,
			trendLabel !== 'Stable trend'
				? `Overall, the series shows an ${trendLabel.toLowerCase()}.`
				: `Overall, the series remains relatively stable.`
		];

		dataNotes = [];
		if (missingCount > 0) {
			dataNotes.push(`Missing ${primaryAxis} values: ${missingCount}`);
		}
		dataNotes.push(`Distinct ${config.xAxis} entries: ${uniqueX}`);
	}

	function shouldShowDataLabels(chartType: string, rowCount: number) {
		if (['pie', 'donut', 'radialBar', 'polarArea'].includes(chartType)) return true;
		if (chartType === 'bar') return rowCount <= 12;
		return rowCount <= 8;
	}

	function buildSeries(
		chartType: string,
		activeYAxes: string[],
		categories: (string | number)[],
		rows: any[]
	) {
		if (['pie', 'donut', 'radialBar', 'polarArea'].includes(chartType)) {
			return rows.map((row) => parseFloat(row[activeYAxes[0]]) || 0);
		}

		if (chartType === 'scatter') {
			return activeYAxes.map((axis) => ({
				name: axis,
				data: rows.map((row) => ({
					x: row[config.xAxis],
					y: parseFloat(row[axis]) || 0
				}))
			}));
		}

		if (chartType === 'heatmap') {
			return activeYAxes.map((axis) => ({
				name: axis,
				data: rows.map((row, index) => ({
					x: categories[index],
					y: parseFloat(row[axis]) || 0
				}))
			}));
		}

		return activeYAxes.map((axis) => ({
			name: axis,
			data: rows.map((row) => parseFloat(row[axis]) || 0)
		}));
	}

	function getOptions() {
		const rows = Array.isArray(data) ? data : [];
		const categories = rows.map((row) => row[config.xAxis]);
		// Determine which axes to use (support multiple or single legacy)
		const activeYAxes = config.yAxes && config.yAxes.length > 0 ? config.yAxes : [config.yAxis];
		const yAxisOptions = activeYAxes.map((axis) => ({
			title: {
				text: axis,
				style: {
					color: '#6B7280',
					fontSize: '12px',
					fontWeight: 600
				}
			},
			labels: {
				style: {
					colors: '#6B7280',
					fontSize: '12px'
				},
				formatter: (val: number) => {
					return typeof val === 'number' ? val.toLocaleString() : val;
				}
			}
		}));

		const options: any = {
			series: [],
			chart: {
				type: config.chartType,
				height: 400,
				stacked: config.stacked || false,
				toolbar: {
					show: true,
					tools: {
						download: true,
						selection: true,
						zoom: true,
						zoomin: true,
						zoomout: true,
						pan: true,
						reset: true
					}
				},
				fontFamily: 'inherit'
			},
			colors: [
				'#008FFB',
				'#00E396',
				'#FEB019',
				'#FF4560',
				'#775DD0',
				'#3F51B5',
				'#03A9F4',
				'#4CAF50',
				'#F9CE1D',
				'#FF9800',
				'#33B2DF',
				'#546E7A'
			],
			dataLabels: {
				enabled: shouldShowDataLabels(config.chartType, rows.length),
				style: {
					fontSize: '12px',
					fontWeight: '600',
					colors: ['#000']
				},
				background: {
					enabled: true,
					foreColor: '#fff',
					borderRadius: 2,
					padding: 4,
					opacity: 0.9,
					borderWidth: 1,
					borderColor: '#fff'
				}
			},
			stroke: {
				curve: 'smooth',
				width:
					config.chartType === 'line' || config.chartType === 'area' || config.chartType === 'radar'
						? 3
						: 0
			},
			xaxis: {
				categories: categories,
				labels: {
					style: {
						colors: '#6B7280',
						fontSize: '12px'
					}
				}
			},
			yaxis: yAxisOptions,
			grid: {
				borderColor: '#E5E7EB',
				strokeDashArray: 4
			},
			theme: {
				mode: 'light'
			},
			plotOptions: {
				pie: {
					dataLabels: {
						offset: -5
					}
				},
				bar: {
					dataLabels: {
						position: 'top'
					}
				}
			},
			annotations: {
				yaxis: (config.annotations || []).map((ann) => ({
					y: ann.value,
					borderColor: ann.color,
					label: {
						borderColor: ann.color,
						style: {
							color: '#fff',
							background: ann.color
						},
						text: ann.label
					}
				}))
			}
		};

		// Types that require single series of values (Pie/Donut/Radial/Polar)
		// We'll use the first Y-Axis for these
		if (['pie', 'donut', 'radialBar', 'polarArea'].includes(config.chartType)) {
			options.series = buildSeries(config.chartType, activeYAxes, categories, rows);
			options.labels = categories;
			// Chart type might need to be explicitly set if different from generic 'pie'
			// However, config.chartType is passed to chart.type above.
			// ApexCharts usually infers correct processing from type.

			// Enable percentage display for these charts
			options.dataLabels.formatter = function (val: number) {
				return val.toFixed(1) + '%';
			};
			options.legend = {
				position: 'bottom',
				horizontalAlign: 'center'
			};
			options.tooltip = {
				y: {
					formatter: function (val: number) {
						return val.toString();
					}
				}
			};
		} else {
			// Multi-series types (Bar, Line, Area, Radar, Scatter, Heatmap)
			options.series = buildSeries(config.chartType, activeYAxes, categories, rows);

			// Show actual values
			options.dataLabels.formatter = function (val: number) {
				return typeof val === 'number' ? val.toLocaleString() : val;
			};
			if (config.chartType === 'bar') {
				options.dataLabels.offsetY = -20;
			}
		}

		return options;
	}

	function updateChart() {
		if (chart) {
			const options = getOptions();
			const configKey = getConfigKey();
			const seriesKey = JSON.stringify(options.series);
			const isConfigChange = configKey !== lastConfigKey;
			const isSeriesChange = seriesKey !== lastSeriesKey;

			if (isConfigChange) {
				chart.updateOptions(options);
			} else if (isSeriesChange) {
				chart.updateSeries(options.series);
			}

			lastConfigKey = configKey;
			lastSeriesKey = seriesKey;
		}
	}

	$: if (config && data) {
		analyzeData();
	}

	async function exportToPDF() {
		if (!chart) return;

		try {
			const { default: jsPDF } = await import('jspdf');
			const base64 = await chart.dataURI();
			const pdf = new jsPDF({
				orientation: 'landscape',
				unit: 'mm',
				format: 'a4'
			});

			pdf.setFontSize(16);
			// Show all Y-Axes in title
			const yAxesLabel = (config.yAxes || [config.yAxis]).join(', ');
			pdf.text(`${config.chartType.toUpperCase()} Chart: ${yAxesLabel} by ${config.xAxis}`, 15, 15);

			const imgData = base64.imgURI;
			pdf.addImage(imgData, 'PNG', 15, 25, 270, 150);

			// Add annotations info if any
			if (config.annotations && config.annotations.length > 0) {
				let yPos = 185;
				pdf.setFontSize(10);
				pdf.text('Annotations:', 15, yPos);
				config.annotations.forEach((ann) => {
					yPos += 5;
					pdf.text(`- ${ann.label}: ${ann.value}`, 20, yPos);
				});
			}

			pdf.setFontSize(10);
			pdf.text(`Generated on: ${new Date().toLocaleString()}`, 15, 195);
			pdf.save(`chart-${Date.now()}.pdf`);
		} catch (error) {
			console.error('Error exporting to PDF:', error);
			alert('Failed to export chart to PDF. Please try again.');
		}
	}

	async function exportToPNG() {
		if (!chart) return;
		try {
			await chart.dataURI().then((uri: any) => {
				const link = document.createElement('a');
				link.href = uri.imgURI;
				link.download = `chart-${Date.now()}.png`;
				link.click();
			});
		} catch (error) {
			console.error('Error exporting to PNG:', error);
		}
	}
</script>

<div class="chart-container">
	<div class="chart-header">
		<h3 class="chart-title">
			{config.chartType.charAt(0).toUpperCase() + config.chartType.slice(1)} Chart
		</h3>
		<div class="export-buttons">
			<button class="export-btn" on:click={exportToPDF} title="Export to PDF">
				<svg class="export-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M7 21h10a2 2 0 002-2V9.414a1 1 0 00-.293-.707l-5.414-5.414A1 1 0 0012.586 3H7a2 2 0 00-2 2v14a2 2 0 002 2z"
					/>
				</svg>
				PDF
			</button>
			<button class="export-btn" on:click={exportToPNG} title="Export to PNG">
				<svg class="export-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"
					/>
				</svg>
				PNG
			</button>
		</div>
	</div>
	<div bind:this={chartElement}></div>

	{#if insights.length > 0}
		<div class="insight-panel">
			<div class="insight-header">
				<h4 class="insight-title">Insight Callouts</h4>
				<span class="insight-subtitle">Auto-generated from your data</span>
			</div>
			<div class="insight-grid">
				{#each insights as insight}
					<div class="insight-card">
						<div class="insight-card-title">{insight.title}</div>
						<div class="insight-card-detail">{insight.detail}</div>
					</div>
				{/each}
			</div>
			{#if dataNotes.length > 0}
				<div class="insight-notes">
					{#each dataNotes as note}
						<span class="insight-note">{note}</span>
					{/each}
				</div>
			{/if}
		</div>
	{/if}

	{#if narrative.length > 0}
		<div class="story-panel">
			<div class="story-header">
				<h4 class="story-title">Story Mode</h4>
				<span class="story-subtitle">A short narrative you can use in reports</span>
			</div>
			<ol class="story-list">
				{#each narrative as line}
					<li>{line}</li>
				{/each}
			</ol>
		</div>
	{/if}
</div>

<style>
	.chart-container {
		width: 100%;
		background-color: white;
		border-radius: 0.75rem;
		box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
		padding: 1.5rem;
		margin-top: 2rem;
	}

	.chart-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 1rem;
		flex-wrap: wrap;
		gap: 1rem;
	}

	.chart-title {
		font-size: 1.25rem;
		font-weight: 600;
		color: #111827;
		margin: 0;
	}

	.export-buttons {
		display: flex;
		gap: 0.5rem;
	}

	.export-btn {
		display: inline-flex;
		align-items: center;
		gap: 0.375rem;
		padding: 0.5rem 0.875rem;
		background-color: #f3f4f6;
		color: #374151;
		border: 1px solid #d1d5db;
		border-radius: 0.375rem;
		font-size: 0.875rem;
		font-weight: 500;
		cursor: pointer;
		transition: all 0.2s;
	}

	.export-btn:hover {
		background-color: #e5e7eb;
		border-color: #9ca3af;
		transform: translateY(-1px);
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
	}

	.export-btn:active {
		transform: translateY(0);
	}

	.export-icon {
		width: 16px;
		height: 16px;
	}

	@media (max-width: 640px) {
		.chart-header {
			flex-direction: column;
			align-items: flex-start;
		}

		.export-buttons {
			width: 100%;
			justify-content: flex-start;
		}
	}

	.insight-panel {
		margin-top: 1.5rem;
		padding: 1rem;
		border-radius: 0.75rem;
		background: linear-gradient(135deg, #f8fafc 0%, #eef2ff 100%);
		border: 1px solid #e2e8f0;
	}

	.insight-header {
		display: flex;
		justify-content: space-between;
		align-items: baseline;
		gap: 1rem;
		margin-bottom: 1rem;
		flex-wrap: wrap;
	}

	.insight-title {
		font-size: 1rem;
		font-weight: 700;
		color: #1e293b;
		margin: 0;
	}

	.insight-subtitle {
		font-size: 0.8125rem;
		color: #64748b;
	}

	.insight-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
		gap: 0.75rem;
	}

	.insight-card {
		background-color: white;
		border-radius: 0.5rem;
		padding: 0.75rem;
		border: 1px solid #e2e8f0;
		box-shadow: 0 2px 6px rgba(15, 23, 42, 0.08);
	}

	.insight-card-title {
		font-size: 0.75rem;
		letter-spacing: 0.08em;
		text-transform: uppercase;
		color: #64748b;
		font-weight: 700;
	}

	.insight-card-detail {
		margin-top: 0.35rem;
		font-size: 0.9rem;
		color: #0f172a;
		font-weight: 600;
	}

	.insight-notes {
		display: flex;
		flex-wrap: wrap;
		gap: 0.5rem;
		margin-top: 0.75rem;
	}

	.insight-note {
		background-color: #e0e7ff;
		color: #3730a3;
		padding: 0.35rem 0.6rem;
		border-radius: 999px;
		font-size: 0.75rem;
		font-weight: 600;
	}

	.story-panel {
		margin-top: 1rem;
		padding: 1rem;
		border-radius: 0.75rem;
		background: linear-gradient(120deg, #fff7ed 0%, #ffedd5 100%);
		border: 1px solid #fed7aa;
	}

	.story-header {
		display: flex;
		justify-content: space-between;
		align-items: baseline;
		gap: 1rem;
		margin-bottom: 0.75rem;
		flex-wrap: wrap;
	}

	.story-title {
		font-size: 1rem;
		font-weight: 700;
		color: #7c2d12;
		margin: 0;
	}

	.story-subtitle {
		font-size: 0.8125rem;
		color: #9a3412;
	}

	.story-list {
		margin: 0;
		padding-left: 1.25rem;
		color: #7c2d12;
		font-size: 0.9rem;
	}
</style>
