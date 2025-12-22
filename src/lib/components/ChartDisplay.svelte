<script lang="ts">
	import { onMount } from 'svelte';
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

	function getOptions() {
		const categories = data.map((row) => row[config.xAxis]);
		// Determine which axes to use (support multiple or single legacy)
		const activeYAxes = config.yAxes && config.yAxes.length > 0 ? config.yAxes : [config.yAxis];

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
				enabled: true,
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
			yaxis: {
				labels: {
					style: {
						colors: '#6B7280',
						fontSize: '12px'
					},
					formatter: (val: number) => {
						return typeof val === 'number' ? val.toLocaleString() : val;
					}
				}
			},
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
			const seriesData = data.map((row) => parseFloat(row[activeYAxes[0]]) || 0);
			options.series = seriesData;
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
			options.series = activeYAxes.map((axis) => ({
				name: axis,
				data: data.map((row) => parseFloat(row[axis]) || 0)
			}));

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
			chart.updateOptions(getOptions());
		}
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
</style>
