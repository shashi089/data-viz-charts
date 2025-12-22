<script lang="ts">
	import FileUpload from '$lib/components/FileUpload.svelte';
	import DataTable from '$lib/components/DataTable.svelte';
	import ChartConfig from '$lib/components/ChartConfig.svelte';
	import ChartDisplay from '$lib/components/ChartDisplay.svelte';

	let data: any[] = [];
	let columns: string[] = [];

	interface ChartConfiguration {
		id: string;
		chartType: string;
		xAxis: string;
		yAxis: string;
		yAxes?: string[];
		stacked?: boolean;
		annotations?: { value: number; label: string; color: string }[];
		filteredData?: any[];
	}

	let chartConfigs: ChartConfiguration[] = [];

	async function handleDataLoaded(event: CustomEvent<any[]>) {
		data = event.detail;
		if (data.length > 0) {
			columns = Object.keys(data[0]);
		}
		chartConfigs = [];
	}

	function handleGenerateChart(
		event: CustomEvent<{
			chartType: string;
			xAxis: string;
			yAxis: string;
			yAxes?: string[];
			stacked?: boolean;
			annotations?: { value: number; label: string; color: string }[];
			filteredData?: any[];
		}>
	) {
		const newChart = {
			...event.detail,
			id: crypto.randomUUID()
		};
		chartConfigs = [...chartConfigs, newChart];

		// Scroll to the new chart
		setTimeout(() => {
			const element = document.getElementById(`chart-${newChart.id}`);
			element?.scrollIntoView({ behavior: 'smooth', block: 'center' });
		}, 100);
	}

	function removeChart(id: string) {
		chartConfigs = chartConfigs.filter((c) => c.id !== id);
	}

	function clearAllCharts() {
		chartConfigs = [];
	}

	function handleReset() {
		data = [];
		columns = [];
		chartConfigs = [];
	}
</script>

<div class="app-container">
	<!-- Header Navigation -->
	<header class="app-header">
		<div class="header-content">
			<div class="header-logo">
				<svg class="header-logo-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M7 12l3-3 3 3 4-4M8 21l4-4 4 4M3 4h18M4 4h16v12a1 1 0 01-1 1H5a1 1 0 01-1-1V4z"
					/>
				</svg>
				<span class="header-logo-text">Data Viz Charts</span>
			</div>

			<a
				href="https://github.com/shashi089"
				target="_blank"
				rel="noopener noreferrer"
				class="header-github"
			>
				<span class="header-name">Shashidhar</span>
				<svg class="header-github-icon" fill="currentColor" viewBox="0 0 24 24">
					<path
						d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"
					/>
				</svg>
			</a>
		</div>
	</header>

	<!-- Hero Section -->
	<div class="hero-section">
		<div class="hero-content">
			<div class="badge">
				<svg class="badge-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"
					/>
				</svg>
				<span>Powerful Data Analysis</span>
			</div>

			<h1 class="hero-title">
				Transform Your Data Into
				<span class="gradient-text">Beautiful Insights</span>
			</h1>

			<p class="hero-description">
				Upload CSV or Excel files and instantly visualize your data with interactive charts. Search,
				filter, and export with ease—no coding required.
			</p>

			<div class="features-grid">
				<div class="feature-item">
					<div class="feature-icon">📊</div>
					<div class="feature-text">Multiple Chart Types</div>
				</div>
				<div class="feature-item">
					<div class="feature-icon">🔍</div>
					<div class="feature-text">Smart Search & Filter</div>
				</div>
				<div class="feature-item">
					<div class="feature-icon">⚡</div>
					<div class="feature-text">Instant Processing</div>
				</div>
				<div class="feature-item">
					<div class="feature-icon">💾</div>
					<div class="feature-text">Export Charts</div>
				</div>
			</div>
		</div>
	</div>

	<!-- Main Content -->
	<div class="content-wrapper">
		<FileUpload on:dataLoaded={handleDataLoaded} on:reset={handleReset} />

		{#if data.length > 0}
			<div class="fade-in">
				<DataTable {data} />
			</div>

			<div class="fade-in delay-1">
				<ChartConfig {columns} {data} on:generateChart={handleGenerateChart} />
			</div>

			{#if chartConfigs.length > 0}
				<div class="charts-section fade-in delay-2">
					<div class="section-header">
						<h2 class="section-title">Visualizations</h2>
						<button class="clear-all-btn" on:click={clearAllCharts}>
							<svg class="clear-all-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
								/>
							</svg>
							Clear All
						</button>
					</div>

					<div class="charts-grid-wrapper">
						{#each chartConfigs as config (config.id)}
							<div class="chart-wrapper" id="chart-{config.id}">
								<div class="chart-actions">
									<button
										class="remove-chart-btn"
										on:click={() => removeChart(config.id)}
										title="Remove Chart"
									>
										<svg
											class="chart-remove-icon"
											fill="none"
											stroke="currentColor"
											viewBox="0 0 24 24"
										>
											<path
												stroke-linecap="round"
												stroke-linejoin="round"
												stroke-width="2"
												d="M6 18L18 6M6 6l12 12"
											/>
										</svg>
									</button>
								</div>
								<ChartDisplay data={config.filteredData || data} {config} />
							</div>
						{/each}
					</div>
				</div>
			{/if}
		{/if}
	</div>

	<!-- Footer -->
	<footer class="footer">
		<div class="footer-content">
			<p class="footer-text">
				Designed by <span class="designer-name">Shashidhar</span>
			</p>
			<a
				href="https://github.com/shashi089"
				target="_blank"
				rel="noopener noreferrer"
				class="github-link"
				aria-label="GitHub Profile"
			>
				<svg
					class="github-icon"
					fill="currentColor"
					viewBox="0 0 24 24"
					xmlns="http://www.w3.org/2000/svg"
				>
					<path
						d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"
					/>
				</svg>
			</a>
		</div>
	</footer>
</div>

<style>
	:global(html) {
		scroll-behavior: smooth;
	}

	:global(body) {
		margin: 0;
		padding: 0;
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		min-height: 100vh;
		font-family:
			-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
	}

	/* Custom Scrollbar */
	:global(::-webkit-scrollbar) {
		width: 8px;
		height: 8px;
	}

	:global(::-webkit-scrollbar-track) {
		background: rgba(255, 255, 255, 0.1);
	}

	:global(::-webkit-scrollbar-thumb) {
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		border-radius: 10px;
		transition: background 0.3s ease;
	}

	:global(::-webkit-scrollbar-thumb:hover) {
		background: linear-gradient(135deg, #764ba2 0%, #667eea 100%);
	}

	/* Firefox Scrollbar */
	:global(*) {
		scrollbar-width: thin;
		scrollbar-color: #667eea rgba(255, 255, 255, 0.1);
	}

	.app-container {
		position: relative;
		min-height: 100vh;
		overflow: hidden;
	}

	/* Header */
	.app-header {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		z-index: 100;
		background: linear-gradient(
			135deg,
			rgba(102, 126, 234, 0.95) 0%,
			rgba(118, 75, 162, 0.95) 100%
		);
		backdrop-filter: blur(20px);
		border-bottom: 1px solid rgba(255, 255, 255, 0.3);
		padding: 1rem 0;
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
	}

	.header-content {
		max-width: 80rem;
		margin: 0 auto;
		padding: 0 1.5rem;
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.header-logo {
		display: flex;
		align-items: center;
		gap: 0.75rem;
		transition: transform 0.3s ease;
	}

	.header-logo:hover {
		transform: scale(1.05);
	}

	.header-logo-icon {
		width: 2rem;
		height: 2rem;
		color: white;
	}

	.header-logo-text {
		font-size: 1.25rem;
		font-weight: 700;
		background: linear-gradient(135deg, #ffeaa7 0%, #ffffff 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
		letter-spacing: -0.02em;
	}

	.header-github {
		display: flex;
		align-items: center;
		gap: 0.75rem;
		background: rgba(255, 255, 255, 0.15);
		backdrop-filter: blur(10px);
		border: 1px solid rgba(255, 255, 255, 0.2);
		padding: 0.5rem 1rem;
		border-radius: 2rem;
		color: white;
		text-decoration: none;
		transition: all 0.3s ease;
	}

	.header-github:hover {
		background: rgba(255, 255, 255, 0.25);
		transform: translateY(-2px);
		box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
	}

	.header-name {
		font-size: 1rem;
		font-weight: 600;
		background: linear-gradient(135deg, #ffeaa7 0%, #ffffff 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
	}

	.header-github-icon {
		width: 1.5rem;
		height: 1.5rem;
		color: white;
		transition: transform 0.3s ease;
	}

	.header-github:hover .header-github-icon {
		transform: rotate(360deg);
	}

	/* Hero Section */
	.hero-section {
		padding: 8rem 1.5rem 3rem;
		text-align: center;
		position: relative;
		z-index: 1;
	}

	.hero-content {
		max-width: 56rem;
		margin: 0 auto;
	}

	.badge {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		background: rgba(255, 255, 255, 0.15);
		backdrop-filter: blur(10px);
		border: 1px solid rgba(255, 255, 255, 0.2);
		padding: 0.5rem 1rem;
		border-radius: 2rem;
		color: white;
		font-size: 0.875rem;
		font-weight: 500;
		margin-bottom: 1.5rem;
	}

	.badge-icon {
		width: 1.25rem;
		height: 1.25rem;
	}

	.hero-title {
		font-size: clamp(2rem, 5vw, 3.5rem);
		font-weight: 800;
		color: white;
		line-height: 1.2;
		margin-bottom: 1.5rem;
		letter-spacing: -0.02em;
	}

	.gradient-text {
		display: block;
		background: linear-gradient(135deg, #ffd89b 0%, #19547b 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
	}

	.hero-description {
		font-size: 1.125rem;
		color: rgba(255, 255, 255, 0.9);
		line-height: 1.7;
		margin-bottom: 2.5rem;
		max-width: 42rem;
		margin-left: auto;
		margin-right: auto;
	}

	.features-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
		gap: 1rem;
		max-width: 40rem;
		margin: 0 auto;
	}

	.feature-item {
		background: rgba(255, 255, 255, 0.1);
		backdrop-filter: blur(10px);
		border: 1px solid rgba(255, 255, 255, 0.2);
		padding: 1rem;
		border-radius: 1rem;
		transition: all 0.3s ease;
	}

	.feature-item:hover {
		transform: translateY(-4px);
		background: rgba(255, 255, 255, 0.15);
		box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
	}

	.feature-icon {
		font-size: 1.75rem;
		margin-bottom: 0.5rem;
	}

	.feature-text {
		color: white;
		font-size: 0.875rem;
		font-weight: 500;
	}

	/* Content Wrapper */
	.content-wrapper {
		max-width: 80rem;
		margin: 0 auto;
		padding: 0 1.5rem 4rem;
		position: relative;
		z-index: 1;
	}

	/* Charts Section */
	.charts-section {
		margin-top: 3rem;
	}

	.section-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 1.5rem;
		padding: 0 0.5rem;
	}

	.section-title {
		font-size: 1.5rem;
		font-weight: 700;
		color: white;
		margin: 0;
	}

	.clear-all-btn {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0.5rem 1rem;
		background-color: rgba(255, 255, 255, 0.1);
		color: white;
		border: 1px solid rgba(255, 255, 255, 0.2);
		border-radius: 0.5rem;
		font-size: 0.875rem;
		font-weight: 500;
		cursor: pointer;
		transition: all 0.2s;
	}

	.clear-all-btn:hover {
		background-color: rgba(239, 68, 68, 0.2);
		border-color: rgba(239, 68, 68, 0.4);
		color: #fca5a5;
	}

	.clear-all-icon {
		width: 1rem;
		height: 1rem;
	}

	.charts-grid-wrapper {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(600px, 1fr));
		gap: 2rem;
	}

	.chart-wrapper {
		position: relative;
		transition: all 0.3s ease;
	}

	.chart-actions {
		position: absolute;
		top: 3rem;
		right: 2rem;
		z-index: 10;
		opacity: 0;
		transition: opacity 0.2s;
	}

	.chart-wrapper:hover .chart-actions {
		opacity: 1;
	}

	.remove-chart-btn {
		padding: 0.5rem;
		background-color: #fee2e2;
		color: #dc2626;
		border: none;
		border-radius: 0.5rem;
		cursor: pointer;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
		transition: all 0.2s;
	}

	.chart-remove-icon {
		width: 1.25rem;
		height: 1.25rem;
	}

	.remove-chart-btn:hover {
		background-color: #fecaca;
		transform: scale(1.1);
	}

	/* Animations */
	.fade-in {
		animation: fadeInUp 0.6s ease-out forwards;
		opacity: 0;
	}

	.delay-1 {
		animation-delay: 0.2s;
	}

	.delay-2 {
		animation-delay: 0.4s;
	}

	@keyframes fadeInUp {
		0% {
			opacity: 0;
			transform: translateY(50px);
		}
		60% {
			opacity: 1;
			transform: translateY(-10px);
		}
		80% {
			transform: translateY(5px);
		}
		100% {
			opacity: 1;
			transform: translateY(0);
		}
	}

	/* Responsive */
	@media (max-width: 768px) {
		.header-content {
			padding: 0 1rem;
		}

		.header-logo-icon {
			width: 1.5rem;
			height: 1.5rem;
		}

		.header-logo-text {
			font-size: 1rem;
		}

		.header-name {
			font-size: 0.875rem;
		}

		.header-github {
			padding: 0.4rem 0.75rem;
			gap: 0.5rem;
		}

		.header-github-icon {
			width: 1.25rem;
			height: 1.25rem;
		}

		.hero-section {
			padding: 7rem 1rem 2rem;
		}

		.features-grid {
			grid-template-columns: repeat(2, 1fr);
		}

		.content-wrapper {
			padding: 0 1rem 3rem;
		}

		.charts-grid-wrapper {
			grid-template-columns: 1fr;
		}

		.chart-actions {
			opacity: 1;
			top: 1rem;
			right: 1rem;
		}
	}

	/* Footer */
	.footer {
		position: relative;
		z-index: 1;
		padding: 2rem 1.5rem;
		margin-top: 4rem;
	}

	.footer-content {
		max-width: 80rem;
		margin: 0 auto;
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 1.5rem;
		background: rgba(255, 255, 255, 0.1);
		backdrop-filter: blur(10px);
		border: 1px solid rgba(255, 255, 255, 0.2);
		padding: 1.5rem 2rem;
		border-radius: 1rem;
		box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
	}

	.footer-text {
		margin: 0;
		color: rgba(255, 255, 255, 0.9);
		font-size: 1rem;
		font-weight: 400;
	}

	.designer-name {
		font-weight: 600;
		background: linear-gradient(135deg, #ffeaa7 0%, #ffffff 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
	}

	.github-link {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 2.5rem;
		height: 2.5rem;
		background: rgba(255, 255, 255, 0.15);
		border: 1px solid rgba(255, 255, 255, 0.2);
		border-radius: 0.5rem;
		color: white;
		transition: all 0.3s ease;
		text-decoration: none;
	}

	.github-link:hover {
		background: rgba(255, 255, 255, 0.25);
		transform: translateY(-2px) scale(1.05);
		box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
	}

	.github-icon {
		width: 1.5rem;
		height: 1.5rem;
		transition: transform 0.3s ease;
	}

	.github-link:hover .github-icon {
		transform: rotate(360deg);
	}

	@media (max-width: 768px) {
		.footer {
			padding: 1.5rem 1rem;
			margin-top: 3rem;
		}

		.footer-content {
			flex-direction: column;
			gap: 1rem;
			padding: 1.25rem 1.5rem;
		}

		.footer-text {
			font-size: 0.875rem;
			text-align: center;
		}
	}
</style>
