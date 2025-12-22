<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<!-- Favicon -->
	<link rel="icon" href={favicon} />

	<!-- Primary Meta Tags -->
	<title
		>Data Viz Charts - Free Online Data Visualization Tool | Create Interactive Charts from CSV &
		Excel</title
	>
	<meta
		name="title"
		content="Data Viz Charts - Free Online Data Visualization Tool | Create Interactive Charts from CSV & Excel"
	/>
	<meta
		name="description"
		content="Transform your data into beautiful, interactive charts instantly. Upload CSV or Excel files and create stunning visualizations with pie charts, bar graphs, line charts, and more. No coding required - 100% free data analysis tool."
	/>
	<meta
		name="keywords"
		content="data visualization, chart maker, CSV to chart, Excel chart creator, free data analysis, interactive charts, pie chart maker, bar chart generator, line graph tool, data analytics, business intelligence, ApexCharts, online chart tool"
	/>
	<meta name="author" content="Shashidhar" />
	<meta name="robots" content="index, follow" />
	<meta name="language" content="English" />
	<meta name="revisit-after" content="7 days" />

	<!-- Viewport for Responsive Design -->
	<meta name="viewport" content="width=device-width, initial-scale=1.0" />

	<!-- Open Graph / Facebook -->
	<meta property="og:type" content="website" />
	<meta property="og:url" content="https://data-viz-charts.netlify.app/" />
	<meta property="og:title" content="Data Viz Charts - Free Online Data Visualization Tool" />
	<meta
		property="og:description"
		content="Create beautiful, interactive charts from your CSV and Excel data. Free online tool with multiple chart types - no coding required!"
	/>
	<meta property="og:image" content="https://data-viz-charts.netlify.app/og-image.png" />
	<meta property="og:site_name" content="Data Viz Charts" />

	<!-- Additional SEO -->
	<meta name="theme-color" content="#667eea" />
	<link rel="canonical" href="https://data-viz-charts.netlify.app/" />

	<!-- Structured Data / JSON-LD -->
	<script type="application/ld+json">
		{
			"@context": "https://schema.org",
			"@type": "WebApplication",
			"name": "Data Viz Charts",
			"description": "Free online data visualization tool to create interactive charts from CSV and Excel files",
			"url": "https://data-viz-charts.netlify.app/",
			"applicationCategory": "BusinessApplication",
			"operatingSystem": "Any",
			"offers": {
				"@type": "Offer",
				"price": "0",
				"priceCurrency": "USD"
			},
			"creator": {
				"@type": "Person",
				"name": "Shashidhar",
				"url": "https://github.com/shashi089"
			},
			"featureList": [
				"Upload CSV and Excel files",
				"Create multiple chart types (Bar, Line, Pie, Donut, Area, Scatter, Heatmap, etc.)",
				"Interactive data filtering",
				"Search and pagination",
				"Export charts as SVG, PNG, or PDF",
				"No coding required",
				"100% free to use"
			]
		}
	</script>
</svelte:head>

{@render children()}
