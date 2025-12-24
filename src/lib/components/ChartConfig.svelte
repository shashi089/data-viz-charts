<script lang="ts">
	import { createEventDispatcher } from 'svelte';

	export let columns: string[] = [];
	export let data: any[] = [];

	const dispatch = createEventDispatcher();

	let chartType = 'bar';
	let xAxis = '';

	// Multi-series state
	let yAxes: string[] = [];
	let selectedYAxis = ''; // Temporary holder for selection
	let stacked = false;

	// Annotation state
	let annotations: { value: number; label: string; color: string }[] = [];
	let newAnnotation = { value: '', label: '', color: '#FF4560' };

	// Filter state
	let filters: { column: string; operator: string; value: string }[] = [];
	let showFilters = false;

	// Chart type information with SVG icons
	const chartTypes = [
		{
			value: 'bar',
			label: 'Bar Chart',
			description: 'Compare categories',
			example: 'Sales by Region',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)',
			icon: 'M3 13h4v8H3v-8zm6-6h4v14H9V7zm6 4h4v10h-4V11z',
			recommended: false
		},
		{
			value: 'column',
			label: 'Column Chart',
			description: 'Vertical comparison',
			example: 'Monthly Revenue',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)',
			icon: 'M3 3v18h18M7 16V9m4 7v-5m4 5V7m4 9v-3',
			recommended: false
		},
		{
			value: 'line',
			label: 'Line Chart',
			description: 'Trends over time',
			example: 'Sales Trend',
			requirements: 'X-Axis: Time/Sequence, Y-Axis: Values (numbers)',
			icon: 'M3 17l3-3 4 4 8-8',
			recommended: true
		},
		{
			value: 'area',
			label: 'Area Chart',
			description: 'Filled trend viz',
			example: 'Cumulative Sales',
			requirements: 'X-Axis: Time/Sequence, Y-Axis: Values (numbers)',
			icon: 'M3 17l3-3 4 4 8-8v11H3z',
			recommended: false
		},
		{
			value: 'pie',
			label: 'Pie Chart',
			description: 'Part-to-whole',
			example: 'Market Share',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)',
			icon: 'M12 2v10l8.66 5A10 10 0 1 1 12 2z',
			recommended: false
		},
		{
			value: 'donut',
			label: 'Donut Chart',
			description: 'Hollow pie chart',
			example: 'Revenue Distribution',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)',
			icon: 'M12 2a10 10 0 1 0 10 10h-6a4 4 0 1 1-4-4V2z',
			recommended: false
		},
		{
			value: 'radar',
			label: 'Radar Chart',
			description: 'Multi-variable',
			example: 'Skill Assessment',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)',
			icon: 'M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z',
			recommended: false
		},
		{
			value: 'scatter',
			label: 'Scatter Plot',
			description: 'Correlation view',
			example: 'Height vs Weight',
			requirements: 'X-Axis: Values (numbers), Y-Axis: Values (numbers)',
			icon: 'M5 5h2v2H5V5zm4 4h2v2H9V9zm6 0h2v2h-2V9zm-4 4h2v2h-2v-2zm6-2h2v2h-2v-2z',
			recommended: false
		},
		{
			value: 'heatmap',
			label: 'Heatmap',
			description: 'Density colors',
			example: 'Activity by Hour',
			requirements: 'X-Axis: Categories, Y-Axis: Categories/Values',
			icon: 'M3 3h6v6H3V3zm0 8h6v6H3v-6zm8-8h6v6h-6V3zm0 8h6v6h-6v-6z',
			recommended: false
		},
		{
			value: 'treemap',
			label: 'Treemap',
			description: 'Nested rectangles',
			example: 'File Sizes',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)',
			icon: 'M3 3h8v8H3V3zm10 0h8v5h-8V3zm0 7h8v11h-8V10zM3 13h8v8H3v-8z',
			recommended: false
		},
		{
			value: 'radialBar',
			label: 'Radial Bar',
			description: 'Circular progress',
			example: 'Goal Progress',
			requirements: 'X-Axis: Categories (text), Y-Axis: Percentage (0-100)',
			icon: 'M12 2a10 10 0 0 1 10 10h-4a6 6 0 0 0-6-6V2z',
			recommended: false
		},
		{
			value: 'polarArea',
			label: 'Polar Area',
			description: 'Variable radius',
			example: 'Seasonal Data',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)',
			icon: 'M12 2l2.5 7.5H22l-6 4.5 2.5 7.5L12 17l-6.5 4.5L8 14 2 9.5h7.5L12 2z',
			recommended: false
		}
	];

	$: selectedChartInfo = chartTypes.find((ct) => ct.value === chartType);
	$: recommendation = getRecommendation();

	function addYAxis() {
		if (selectedYAxis && !yAxes.includes(selectedYAxis)) {
			yAxes = [...yAxes, selectedYAxis];
			selectedYAxis = '';
		}
	}

	function removeYAxis(index: number) {
		yAxes = yAxes.filter((_, i) => i !== index);
	}

	function addAnnotation() {
		if (newAnnotation.value && newAnnotation.label) {
			annotations = [
				...annotations,
				{
					value: parseFloat(newAnnotation.value),
					label: newAnnotation.label,
					color: newAnnotation.color
				}
			];
			newAnnotation = { value: '', label: '', color: '#FF4560' };
		}
	}

	function removeAnnotation(index: number) {
		annotations = annotations.filter((_, i) => i !== index);
	}

	function addFilter() {
		filters = [...filters, { column: columns[0] || '', operator: 'equals', value: '' }];
	}

	function removeFilter(index: number) {
		filters = filters.filter((_, i) => i !== index);
	}

	function toggleFilters() {
		showFilters = !showFilters;
	}

	function applyFilters(data: any[]) {
		if (filters.length === 0) return data;

		return data.filter((row) => {
			return filters.every((filter) => {
				if (!filter.column || !filter.value) return true;

				const cellValue = String(row[filter.column]).toLowerCase();
				const filterValue = filter.value.toLowerCase();

				switch (filter.operator) {
					case 'equals':
						return cellValue === filterValue;
					case 'contains':
						return cellValue.includes(filterValue);
					case 'starts_with':
						return cellValue.startsWith(filterValue);
					case 'ends_with':
						return cellValue.endsWith(filterValue);
					case 'greater_than':
						return parseFloat(cellValue) > parseFloat(filterValue);
					case 'less_than':
						return parseFloat(cellValue) < parseFloat(filterValue);
					default:
						return true;
				}
			});
		});
	}

	function handleSubmit() {
		if (xAxis && yAxes.length > 0) {
			const filteredData = applyFilters(data);
			dispatch('generateChart', {
				chartType,
				xAxis,
				yAxis: yAxes[0], // Keep for backward compatibility/single series logic if needed
				yAxes,
				stacked,
				annotations,
				filteredData
			});
		}
	}

	$: filteredCount = applyFilters(data).length;

	function sampleValues(column: string, limit = 50) {
		return data
			.slice(0, limit)
			.map((row) => row[column])
			.filter((val) => val !== null && val !== undefined && val !== '');
	}

	function isNumericColumn(column: string) {
		const values = sampleValues(column);
		if (values.length === 0) return false;
		return values.every((val) => Number.isFinite(Number(val)));
	}

	function isDateColumn(column: string) {
		const values = sampleValues(column);
		if (values.length === 0) return false;
		return values.every((val) => !Number.isNaN(Date.parse(String(val))));
	}

	function uniqueCount(column: string) {
		return new Set(data.map((row) => row[column])).size;
	}

	function getRecommendation() {
		if (!xAxis || yAxes.length === 0) return null;

		const xIsDate = isDateColumn(xAxis);
		const xIsNumeric = isNumericColumn(xAxis);
		const yIsNumeric = yAxes.every((axis) => isNumericColumn(axis));
		const uniqueX = uniqueCount(xAxis);
		const fewCategories = uniqueX > 0 && uniqueX <= 6;

		let value = 'bar';
		let reason = 'Good for category comparisons.';

		if (xIsDate && yIsNumeric) {
			value = 'line';
			reason = 'Time-like X axis suggests a trend line.';
		} else if (xIsNumeric && yIsNumeric && yAxes.length === 1) {
			value = 'scatter';
			reason = 'Two numeric axes are best for correlation.';
		} else if (yAxes.length > 1) {
			value = 'line';
			reason = 'Multiple series read well as lines.';
		} else if (fewCategories && yIsNumeric) {
			value = 'pie';
			reason = 'Few categories work well as proportions.';
		}

		const label = chartTypes.find((ct) => ct.value === value)?.label ?? value;

		return { value, label, reason };
	}
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
	<link
		href="https://fonts.googleapis.com/css2?family=Outfit:wght@400;500;600;700;800&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div class="config-container">
	<h2 class="config-title">Chart Configuration</h2>

	<!-- SECTION 1: PICK A CHART (Visual Picker) -->
	<div class="chart-picker-section">
		<div class="section-header">
			<h3 class="section-title">Pick a Chart</h3>
			<p class="section-subtitle">Choose the visualization that best tells your story</p>
		</div>

		<!-- Sticky Recommendation Chip -->
		{#if recommendation}
			<div class="recommendation-sticky">
				<div class="recommendation-content">
					<svg class="recommendation-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z"
						/>
					</svg>
					<div class="recommendation-text-block">
						<span class="recommendation-label">Suggested:</span>
						<span class="recommendation-value">{recommendation.label}</span>
						<span class="recommendation-reason">{recommendation.reason}</span>
					</div>
				</div>
				<button
					class="recommendation-apply-btn"
					on:click={() => (chartType = recommendation.value)}
					disabled={chartType === recommendation.value}
				>
					{chartType === recommendation.value ? '✓ Applied' : 'Apply'}
				</button>
			</div>
		{/if}

		<!-- Chart Type Cards Grid -->
		<div class="chart-cards-grid">
			{#each chartTypes as type}
				<button
					class="chart-card"
					class:selected={chartType === type.value}
					on:click={() => (chartType = type.value)}
				>
					{#if type.recommended}
						<div class="recommended-badge">Recommended</div>
					{/if}
					<div class="chart-icon-wrapper">
						<svg class="chart-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d={type.icon} />
						</svg>
					</div>
					<div class="chart-card-content">
						<h4 class="chart-card-title">{type.label}</h4>
						<p class="chart-card-description">{type.description}</p>
						<p class="chart-card-example">{type.example}</p>
					</div>
					<div class="chart-card-checkmark">
						<svg fill="currentColor" viewBox="0 0 20 20">
							<path
								fill-rule="evenodd"
								d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z"
								clip-rule="evenodd"
							/>
						</svg>
					</div>
				</button>
			{/each}
		</div>
	</div>

	<!-- SECTION 2: TUNE DATA (Axes & Filters) -->
	<div class="data-tune-section">
		<div class="section-header">
			<h3 class="section-title">Tune Data</h3>
			<p class="section-subtitle">Configure axes and apply filters</p>
		</div>

		<div class="config-grid">
			<div class="config-field">
				<label for="xAxis" class="config-label">X-Axis (Category/Label)</label>
				<select id="xAxis" bind:value={xAxis} class="config-select">
					<option value="" disabled selected>Select Column</option>
					{#each columns as col}
						<option value={col}>{col}</option>
					{/each}
				</select>
			</div>

			<div class="config-field">
				<label for="yAxisSelect" class="config-label">Y-Axis (Values)</label>
				<div class="input-group">
					<select id="yAxisSelect" bind:value={selectedYAxis} class="config-select">
						<option value="" disabled selected>Select Column</option>
						{#each columns as col}
							<option value={col}>{col}</option>
						{/each}
					</select>
					<button
						class="add-btn"
						on:click={addYAxis}
						disabled={!selectedYAxis}
						aria-label="Add Y-Axis"
					>
						<svg class="add-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M12 4v16m8-8H4"
							/>
						</svg>
					</button>
				</div>

				{#if yAxes.length > 0}
					<div class="tags-container">
						{#each yAxes as axis, i}
							<div class="tag">
								{axis}
								<button class="tag-remove" on:click={() => removeYAxis(i)} aria-label="Remove axis"
									>×</button
								>
							</div>
						{/each}
					</div>
				{/if}

				{#if ['bar', 'column', 'area', 'line'].includes(chartType)}
					<div class="checkbox-container">
						<label class="checkbox-label">
							<input type="checkbox" bind:checked={stacked} />
							<span>Stacked Chart</span>
						</label>
					</div>
				{/if}
			</div>
		</div>

		<!-- Annotations Section -->
		<div class="annotations-section">
			<h3 class="subsection-title">Threshold Lines</h3>
			<div class="annotation-form">
				<input
					type="number"
					bind:value={newAnnotation.value}
					placeholder="Value"
					class="filter-input"
				/>
				<input
					type="text"
					bind:value={newAnnotation.label}
					placeholder="Label"
					class="filter-input"
				/>
				<input type="color" bind:value={newAnnotation.color} class="color-input" title="Color" />
				<button class="add-filter-btn" on:click={addAnnotation}>Add Line</button>
			</div>

			{#if annotations.length > 0}
				<div class="tags-container">
					{#each annotations as ann, i}
						<div class="tag tag-annotation" style="border-left: 3px solid {ann.color}">
							{ann.label}: {ann.value}
							<button
								class="tag-remove"
								on:click={() => removeAnnotation(i)}
								aria-label="Remove annotation">×</button
							>
						</div>
					{/each}
				</div>
			{/if}
		</div>

		<!-- Filter Section -->
		<div class="filter-section">
			<button class="filter-toggle-btn" on:click={toggleFilters}>
				<svg class="filter-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M3 4a1 1 0 011-1h16a1 1 0 011 1v2.586a1 1 0 01-.293.707l-6.414 6.414a1 1 0 00-.293.707V17l-4 4v-6.586a1 1 0 00-.293-.707L3.293 7.293A1 1 0 013 6.586V4z"
					/>
				</svg>
				{showFilters ? 'Hide Filters' : 'Add Filters'}
				{#if filters.length > 0}
					<span class="filter-badge">{filters.length}</span>
				{/if}
			</button>

			{#if showFilters}
				<div class="filters-container">
					{#each filters as filter, index}
						<div class="filter-row">
							<select bind:value={filter.column} class="filter-select">
								{#each columns as col}
									<option value={col}>{col}</option>
								{/each}
							</select>

							<select bind:value={filter.operator} class="filter-select">
								<option value="equals">Equals</option>
								<option value="contains">Contains</option>
								<option value="starts_with">Starts With</option>
								<option value="ends_with">Ends With</option>
								<option value="greater_than">Greater Than</option>
								<option value="less_than">Less Than</option>
							</select>

							<input
								type="text"
								bind:value={filter.value}
								placeholder="Filter value..."
								class="filter-input"
							/>

							<button
								class="remove-filter-btn"
								on:click={() => removeFilter(index)}
								aria-label="Remove filter"
							>
								<svg class="remove-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										stroke-width="2"
										d="M6 18L18 6M6 6l12 12"
									/>
								</svg>
							</button>
						</div>
					{/each}

					<button class="add-filter-btn" on:click={addFilter}>
						<svg class="add-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M12 4v16m8-8H4"
							/>
						</svg>
						Add Filter
					</button>

					{#if filters.length > 0}
						<div class="filter-info">
							<svg class="info-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
								/>
							</svg>
							Filtered: {filteredCount} of {data.length} records
						</div>
					{/if}
				</div>
			{/if}
		</div>
	</div>

	<div class="config-actions">
		<button on:click={handleSubmit} disabled={!xAxis || yAxes.length === 0} class="submit-btn">
			Generate Chart
		</button>
	</div>
</div>

<style>
	.config-container {
		width: 100%;
		background-color: white;
		border-radius: 0.75rem;
		box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
		padding: 1.5rem;
		margin-top: 2rem;
	}

	.config-title {
		font-size: 1.5rem;
		font-weight: 700;
		color: #111827;
		margin-bottom: 2rem;
		font-family: 'Outfit', sans-serif;
	}

	/* ===== CHART PICKER SECTION ===== */
	.chart-picker-section {
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		border-radius: 1rem;
		padding: 2rem;
		margin-bottom: 2rem;
		position: relative;
		overflow: hidden;
	}

	.chart-picker-section::before {
		content: '';
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background-image:
			radial-gradient(circle at 20% 50%, rgba(255, 255, 255, 0.1) 0%, transparent 50%),
			radial-gradient(circle at 80% 80%, rgba(255, 255, 255, 0.1) 0%, transparent 50%);
		pointer-events: none;
	}

	.section-header {
		position: relative;
		z-index: 1;
		margin-bottom: 1.5rem;
	}

	.section-title {
		font-family: 'Outfit', sans-serif;
		font-size: 1.75rem;
		font-weight: 800;
		color: white;
		margin-bottom: 0.5rem;
		letter-spacing: -0.02em;
	}

	.section-subtitle {
		font-family: 'Outfit', sans-serif;
		font-size: 0.95rem;
		color: rgba(255, 255, 255, 0.9);
		font-weight: 400;
	}

	/* Sticky Recommendation Chip */
	.recommendation-sticky {
		position: relative;
		z-index: 1;
		background: linear-gradient(120deg, #fef3c7 0%, #fde68a 100%);
		border-radius: 0.75rem;
		padding: 1rem 1.25rem;
		margin-bottom: 1.5rem;
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 1rem;
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
		border: 2px solid #f59e0b;
		animation: slideIn 0.4s ease-out;
	}

	@keyframes slideIn {
		from {
			opacity: 0;
			transform: translateY(-10px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	.recommendation-content {
		display: flex;
		align-items: center;
		gap: 0.75rem;
		flex: 1;
	}

	.recommendation-icon {
		width: 24px;
		height: 24px;
		color: #b45309;
		flex-shrink: 0;
	}

	.recommendation-text-block {
		display: flex;
		flex-direction: column;
		gap: 0.125rem;
	}

	.recommendation-label {
		font-family: 'Outfit', sans-serif;
		font-weight: 700;
		font-size: 0.75rem;
		text-transform: uppercase;
		letter-spacing: 0.05em;
		color: #92400e;
	}

	.recommendation-value {
		font-family: 'Outfit', sans-serif;
		font-weight: 700;
		font-size: 1rem;
		color: #7c2d12;
	}

	.recommendation-reason {
		font-family: 'Outfit', sans-serif;
		font-size: 0.8125rem;
		color: #92400e;
		font-weight: 500;
	}

	.recommendation-apply-btn {
		padding: 0.5rem 1.25rem;
		border-radius: 999px;
		border: none;
		background-color: #b45309;
		color: white;
		font-family: 'Outfit', sans-serif;
		font-size: 0.875rem;
		font-weight: 600;
		cursor: pointer;
		transition: all 0.2s ease;
		white-space: nowrap;
		box-shadow: 0 2px 8px rgba(180, 83, 9, 0.3);
	}

	.recommendation-apply-btn:hover:not(:disabled) {
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(180, 83, 9, 0.4);
		background-color: #92400e;
	}

	.recommendation-apply-btn:disabled {
		opacity: 0.7;
		cursor: not-allowed;
		background-color: #059669;
	}

	/* Chart Cards Grid */
	.chart-cards-grid {
		position: relative;
		z-index: 1;
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
		gap: 1rem;
	}

	.chart-card {
		position: relative;
		background: white;
		border-radius: 0.75rem;
		padding: 1.25rem 1rem;
		border: 2px solid transparent;
		cursor: pointer;
		transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
		display: flex;
		flex-direction: column;
		align-items: center;
		text-align: center;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
		overflow: hidden;
	}

	.chart-card:hover {
		transform: translateY(-4px);
		box-shadow: 0 8px 24px rgba(102, 126, 234, 0.25);
		border-color: #667eea;
	}

	.chart-card.selected {
		border-color: #667eea;
		background: linear-gradient(135deg, #f0f4ff 0%, #e0e7ff 100%);
		box-shadow: 0 8px 24px rgba(102, 126, 234, 0.35);
		transform: translateY(-4px);
	}

	.recommended-badge {
		position: absolute;
		top: 0.5rem;
		right: 0.5rem;
		background: linear-gradient(135deg, #10b981 0%, #059669 100%);
		color: white;
		font-family: 'Outfit', sans-serif;
		font-size: 0.625rem;
		font-weight: 700;
		padding: 0.25rem 0.5rem;
		border-radius: 999px;
		text-transform: uppercase;
		letter-spacing: 0.05em;
		box-shadow: 0 2px 6px rgba(16, 185, 129, 0.4);
	}

	.chart-icon-wrapper {
		width: 48px;
		height: 48px;
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		border-radius: 0.5rem;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 0.75rem;
		transition: transform 0.3s ease;
	}

	.chart-card:hover .chart-icon-wrapper {
		transform: scale(1.1) rotate(5deg);
	}

	.chart-card.selected .chart-icon-wrapper {
		background: linear-gradient(135deg, #4f46e5 0%, #6366f1 100%);
	}

	.chart-icon {
		width: 28px;
		height: 28px;
		color: white;
	}

	.chart-card-content {
		flex: 1;
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
	}

	.chart-card-title {
		font-family: 'Outfit', sans-serif;
		font-size: 0.9375rem;
		font-weight: 700;
		color: #111827;
		margin-bottom: 0.25rem;
	}

	.chart-card-description {
		font-family: 'Outfit', sans-serif;
		font-size: 0.75rem;
		color: #6b7280;
		font-weight: 500;
		line-height: 1.3;
	}

	.chart-card-example {
		font-family: 'Outfit', sans-serif;
		font-size: 0.6875rem;
		color: #9ca3af;
		font-style: italic;
		margin-top: 0.25rem;
	}

	.chart-card-checkmark {
		position: absolute;
		top: 0.5rem;
		left: 0.5rem;
		width: 20px;
		height: 20px;
		background-color: #667eea;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		opacity: 0;
		transform: scale(0);
		transition: all 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55);
	}

	.chart-card.selected .chart-card-checkmark {
		opacity: 1;
		transform: scale(1);
	}

	.chart-card-checkmark svg {
		width: 12px;
		height: 12px;
		color: white;
	}

	/* ===== DATA TUNE SECTION ===== */
	.data-tune-section {
		background-color: #f9fafb;
		border-radius: 1rem;
		padding: 2rem;
		border: 2px solid #e5e7eb;
	}

	.data-tune-section .section-title {
		font-family: 'Outfit', sans-serif;
		font-size: 1.5rem;
		font-weight: 700;
		color: #111827;
	}

	.data-tune-section .section-subtitle {
		font-family: 'Outfit', sans-serif;
		color: #6b7280;
	}

	.config-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
		gap: 1.5rem;
		margin-top: 1.5rem;
	}

	.config-field {
		display: flex;
		flex-direction: column;
	}

	.config-label {
		display: block;
		font-size: 0.875rem;
		font-weight: 600;
		color: #374151;
		margin-bottom: 0.5rem;
		font-family: 'Outfit', sans-serif;
	}

	.config-select {
		width: 100%;
		border-radius: 0.5rem;
		border: 2px solid #e5e7eb;
		background-color: white;
		color: #111827;
		padding: 0.75rem 2.5rem 0.75rem 0.875rem;
		font-size: 0.875rem;
		font-weight: 500;
		background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='%23667eea'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2.5' d='M19 9l-7 7-7-7'%3E%3C/path%3E%3C/svg%3E");
		background-repeat: no-repeat;
		background-position: right 0.75rem center;
		background-size: 1.25rem;
		appearance: none;
		cursor: pointer;
		transition: all 0.3s ease;
		box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
	}

	.config-select:hover {
		border-color: #667eea;
		background-color: #f9fafb;
		box-shadow: 0 4px 6px rgba(102, 126, 234, 0.1);
		transform: translateY(-1px);
	}

	.config-select:focus {
		outline: none;
		border-color: #667eea;
		box-shadow: 0 0 0 4px rgba(102, 126, 234, 0.15);
		background-color: white;
	}

	.config-select option {
		cursor: pointer;
		padding: 0.625rem;
	}

	.input-group {
		display: flex;
		gap: 0.5rem;
	}

	.add-btn {
		background-color: #3b82f6;
		color: white;
		border: none;
		border-radius: 0.5rem;
		padding: 0 0.75rem;
		cursor: pointer;
		transition: background-color 0.2s;
	}

	.add-btn:hover:not(:disabled) {
		background-color: #2563eb;
	}

	.add-btn:disabled {
		background-color: #d1d5db;
		cursor: not-allowed;
	}

	.tags-container {
		display: flex;
		flex-wrap: wrap;
		gap: 0.5rem;
		margin-top: 0.75rem;
	}

	.tag {
		background-color: #eff6ff;
		color: #1e40af;
		padding: 0.25rem 0.5rem;
		border-radius: 0.375rem;
		font-size: 0.875rem;
		display: flex;
		align-items: center;
		gap: 0.375rem;
		border: 1px solid #bfdbfe;
	}

	.tag-annotation {
		background-color: #fff1f2;
		color: #be123c;
		border-color: #fecdd3;
	}

	.tag-remove {
		background: none;
		border: none;
		color: inherit;
		cursor: pointer;
		font-size: 1.125rem;
		padding: 0;
		line-height: 1;
		opacity: 0.6;
	}

	.tag-remove:hover {
		opacity: 1;
	}

	.checkbox-container {
		margin-top: 0.75rem;
	}

	.checkbox-label {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		font-size: 0.875rem;
		color: #374151;
		cursor: pointer;
	}

	.annotations-section {
		margin-top: 1.5rem;
		padding-top: 1.5rem;
		border-top: 1px solid #e5e7eb;
	}

	.subsection-title {
		font-size: 1rem;
		font-weight: 600;
		color: #374151;
		margin-bottom: 0.75rem;
		font-family: 'Outfit', sans-serif;
	}

	.annotation-form {
		display: grid;
		grid-template-columns: 1fr 1fr auto auto;
		gap: 0.75rem;
		align-items: center;
	}

	.color-input {
		height: 42px;
		width: 50px;
		padding: 2px;
		border: 1px solid #d1d5db;
		border-radius: 0.375rem;
		cursor: pointer;
	}

	/* Filter Section */
	.filter-section {
		margin-top: 1.5rem;
		padding-top: 1.5rem;
		border-top: 1px solid #e5e7eb;
	}

	.filter-toggle-btn {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0.5rem 1rem;
		background-color: #f3f4f6;
		color: #374151;
		border: 1px solid #d1d5db;
		border-radius: 0.5rem;
		font-size: 0.875rem;
		font-weight: 500;
		cursor: pointer;
		transition: all 0.2s;
	}

	.filter-toggle-btn:hover {
		background-color: #e5e7eb;
	}

	.filter-icon {
		width: 18px;
		height: 18px;
	}

	.filter-badge {
		background-color: #3b82f6;
		color: white;
		padding: 0.125rem 0.5rem;
		border-radius: 9999px;
		font-size: 0.75rem;
		font-weight: 600;
	}

	.filters-container {
		margin-top: 1rem;
		padding: 1rem;
		background-color: white;
		border-radius: 0.5rem;
		border: 1px solid #e5e7eb;
	}

	.filter-row {
		display: grid;
		grid-template-columns: 1fr 1fr 1.5fr auto;
		gap: 0.75rem;
		margin-bottom: 0.75rem;
		align-items: center;
	}

	.filter-select {
		padding: 0.625rem 2rem 0.625rem 0.75rem;
		border: 2px solid #e5e7eb;
		border-radius: 0.5rem;
		font-size: 0.875rem;
		background-color: white;
		background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='%23667eea'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2.5' d='M19 9l-7 7-7-7'%3E%3C/path%3E%3C/svg%3E");
		background-repeat: no-repeat;
		background-position: right 0.5rem center;
		background-size: 1.125rem;
		appearance: none;
		cursor: pointer;
		transition: all 0.2s ease;
		font-weight: 500;
		color: #374151;
	}

	.filter-select:hover {
		border-color: #667eea;
		background-color: #f9fafb;
		box-shadow: 0 2px 4px rgba(102, 126, 234, 0.1);
		transform: translateY(-1px);
	}

	.filter-select:focus {
		outline: none;
		border-color: #667eea;
		box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.15);
		background-color: white;
	}

	.filter-select option {
		cursor: pointer;
		padding: 0.5rem;
	}

	.filter-input {
		padding: 0.5rem;
		border: 1px solid #d1d5db;
		border-radius: 0.375rem;
		font-size: 0.875rem;
	}

	.filter-input:focus {
		outline: none;
		border-color: #3b82f6;
		box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
	}

	.remove-filter-btn {
		padding: 0.5rem;
		background-color: #fee2e2;
		color: #dc2626;
		border: none;
		border-radius: 0.375rem;
		cursor: pointer;
		transition: background-color 0.2s;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.remove-filter-btn:hover {
		background-color: #fecaca;
	}

	.remove-icon {
		width: 16px;
		height: 16px;
	}

	.add-filter-btn {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0.5rem 1rem;
		background-color: white;
		color: #3b82f6;
		border: 1px dashed #3b82f6;
		border-radius: 0.375rem;
		font-size: 0.875rem;
		font-weight: 500;
		cursor: pointer;
		transition: all 0.2s;
		margin-top: 0.5rem;
	}

	.add-filter-btn:hover {
		background-color: #eff6ff;
	}

	.add-icon {
		width: 16px;
		height: 16px;
	}

	.filter-info {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		margin-top: 1rem;
		padding: 0.75rem;
		background-color: #dbeafe;
		border-radius: 0.375rem;
		color: #1e40af;
		font-size: 0.875rem;
		font-weight: 500;
	}

	.info-icon {
		width: 18px;
		height: 18px;
	}

	.config-actions {
		margin-top: 1.5rem;
		display: flex;
		justify-content: flex-end;
	}

	.submit-btn {
		padding: 0.75rem 2rem;
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		color: white;
		font-weight: 600;
		border-radius: 0.5rem;
		box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
		transition: all 0.3s ease;
		border: none;
		cursor: pointer;
		font-size: 0.9375rem;
		font-family: 'Outfit', sans-serif;
	}

	.submit-btn:hover:not(:disabled) {
		transform: translateY(-2px);
		box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
	}

	.submit-btn:disabled {
		opacity: 0.5;
		cursor: not-allowed;
	}

	/* Responsive */
	@media (max-width: 768px) {
		.chart-cards-grid {
			grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
		}

		.filter-row {
			grid-template-columns: 1fr;
		}

		.remove-filter-btn {
			width: 100%;
		}

		.annotation-form {
			grid-template-columns: 1fr 1fr;
		}

		.annotation-form .add-filter-btn {
			grid-column: 1 / -1;
		}

		.recommendation-sticky {
			flex-direction: column;
			align-items: stretch;
		}

		.recommendation-apply-btn {
			width: 100%;
		}
	}
</style>
