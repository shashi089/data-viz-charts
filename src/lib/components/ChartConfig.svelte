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

	// Chart type information
	const chartTypes = [
		{
			value: 'bar',
			label: 'Bar Chart',
			description: 'Compare values across categories',
			example: 'Sales by Region, Products by Category',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)'
		},
		{
			value: 'column',
			label: 'Column Chart',
			description: 'Vertical bars for category comparison',
			example: 'Monthly Revenue, Department Expenses',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)'
		},
		{
			value: 'line',
			label: 'Line Chart',
			description: 'Show trends over time or sequence',
			example: 'Sales Trend, Temperature Over Time',
			requirements: 'X-Axis: Time/Sequence, Y-Axis: Values (numbers)'
		},
		{
			value: 'area',
			label: 'Area Chart',
			description: 'Line chart with filled area below',
			example: 'Cumulative Sales, Stock Price History',
			requirements: 'X-Axis: Time/Sequence, Y-Axis: Values (numbers)'
		},
		{
			value: 'pie',
			label: 'Pie Chart',
			description: 'Show proportions of a whole',
			example: 'Market Share, Budget Allocation',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)'
		},
		{
			value: 'donut',
			label: 'Donut Chart',
			description: 'Pie chart with center hollow',
			example: 'Revenue Distribution, Survey Results',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)'
		},
		{
			value: 'radar',
			label: 'Radar Chart',
			description: 'Compare multiple variables',
			example: 'Skill Assessment, Product Comparison',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)'
		},
		{
			value: 'scatter',
			label: 'Scatter Plot',
			description: 'Show correlation between two variables',
			example: 'Height vs Weight, Price vs Quality',
			requirements: 'X-Axis: Values (numbers), Y-Axis: Values (numbers)'
		},
		{
			value: 'heatmap',
			label: 'Heatmap',
			description: 'Show data density with colors',
			example: 'Activity by Hour/Day, Correlation Matrix',
			requirements: 'X-Axis: Categories, Y-Axis: Categories/Values'
		},
		{
			value: 'treemap',
			label: 'Treemap',
			description: 'Hierarchical data with nested rectangles',
			example: 'File Sizes, Budget Breakdown',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)'
		},
		{
			value: 'radialBar',
			label: 'Radial Bar',
			description: 'Circular progress indicators',
			example: 'Goal Progress, Completion Rates',
			requirements: 'X-Axis: Categories (text), Y-Axis: Percentage (0-100)'
		},
		{
			value: 'polarArea',
			label: 'Polar Area',
			description: 'Pie chart with variable radius',
			example: 'Seasonal Data, Directional Analysis',
			requirements: 'X-Axis: Categories (text), Y-Axis: Values (numbers)'
		}
	];

	$: selectedChartInfo = chartTypes.find((ct) => ct.value === chartType);

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
</script>

<div class="config-container">
	<h2 class="config-title">Chart Configuration</h2>

	<div class="config-grid">
		<div class="config-field full-width">
			<label for="chartType" class="config-label">Chart Type</label>
			<select id="chartType" bind:value={chartType} class="config-select">
				{#each chartTypes as type}
					<option value={type.value}>{type.label}</option>
				{/each}
			</select>
			{#if selectedChartInfo}
				<div class="chart-info">
					<div class="info-row">
						<svg class="info-icon-small" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
							/>
						</svg>
						<span class="info-text">{selectedChartInfo.description}</span>
					</div>
					<div class="info-row">
						<svg class="info-icon-small" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z"
							/>
						</svg>
						<span class="info-text"><strong>Example:</strong> {selectedChartInfo.example}</span>
					</div>
					<div class="info-row requirements">
						<svg class="info-icon-small" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"
							/>
						</svg>
						<span class="info-text"
							><strong>Requirements:</strong> {selectedChartInfo.requirements}</span
						>
					</div>
				</div>
			{/if}
		</div>

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
		font-size: 1.25rem;
		font-weight: 600;
		color: #111827;
		margin-bottom: 1.5rem;
	}

	.config-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
		gap: 1.5rem;
	}

	.full-width {
		grid-column: 1 / -1;
	}

	.config-field {
		display: flex;
		flex-direction: column;
	}

	.config-label {
		display: block;
		font-size: 0.875rem;
		font-weight: 500;
		color: #374151;
		margin-bottom: 0.5rem;
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

	.chart-info {
		margin-top: 0.75rem;
		padding: 1rem;
		background: linear-gradient(135deg, #eff6ff 0%, #dbeafe 100%);
		border-radius: 0.5rem;
		border-left: 4px solid #3b82f6;
	}

	.info-row {
		display: flex;
		align-items: flex-start;
		gap: 0.5rem;
		margin-bottom: 0.5rem;
	}

	.info-row:last-child {
		margin-bottom: 0;
	}

	.info-row.requirements {
		background-color: rgba(255, 255, 255, 0.6);
		padding: 0.5rem;
		border-radius: 0.375rem;
		margin-top: 0.5rem;
	}

	.info-icon-small {
		width: 16px;
		height: 16px;
		color: #3b82f6;
		flex-shrink: 0;
		margin-top: 2px;
	}

	.info-text {
		font-size: 0.8125rem;
		color: #1e40af;
		line-height: 1.5;
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
		background-color: #f9fafb;
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
		padding: 0.625rem 1.5rem;
		background-color: #2563eb;
		color: white;
		font-weight: 500;
		border-radius: 0.5rem;
		box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
		transition: background-color 0.2s;
		border: none;
		cursor: pointer;
		font-size: 0.875rem;
	}

	.submit-btn:hover:not(:disabled) {
		background-color: #1d4ed8;
	}

	.submit-btn:disabled {
		opacity: 0.5;
		cursor: not-allowed;
	}

	/* Responsive */
	@media (max-width: 768px) {
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
	}
</style>
