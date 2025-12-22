<script lang="ts">
	export let data: any[] = [];

	let searchQuery = '';
	let currentPage = 1;
	let pageSize = 10;
	let sortColumn = '';
	let sortDirection: 'asc' | 'desc' = 'asc';

	$: filteredData = data.filter((row) => {
		if (!searchQuery) return true;
		return Object.values(row).some((val) =>
			String(val).toLowerCase().includes(searchQuery.toLowerCase())
		);
	});

	$: sortedData = [...filteredData].sort((a, b) => {
		if (!sortColumn) return 0;
		const aVal = a[sortColumn];
		const bVal = b[sortColumn];

		if (aVal < bVal) return sortDirection === 'asc' ? -1 : 1;
		if (aVal > bVal) return sortDirection === 'asc' ? 1 : -1;
		return 0;
	});

	$: totalPages = Math.ceil(sortedData.length / pageSize);
	$: paginatedData = sortedData.slice((currentPage - 1) * pageSize, currentPage * pageSize);

	function handleSort(column: string) {
		if (sortColumn === column) {
			sortDirection = sortDirection === 'asc' ? 'desc' : 'asc';
		} else {
			sortColumn = column;
			sortDirection = 'asc';
		}
	}

	function goToPage(page: number) {
		if (page >= 1 && page <= totalPages) {
			currentPage = page;
		}
	}
</script>

<div class="table-container">
	<div class="table-header">
		<div class="search-box">
			<input type="text" placeholder="Search..." bind:value={searchQuery} class="search-input" />
		</div>

		<div class="page-size-selector">
			<span>Show</span>
			<select bind:value={pageSize} class="select-input">
				<option value={5}>5</option>
				<option value={10}>10</option>
				<option value={20}>20</option>
				<option value={50}>50</option>
			</select>
			<span>entries</span>
		</div>
	</div>

	<div class="table-wrapper">
		<table class="data-table">
			<thead>
				<tr>
					{#if data.length > 0}
						{#each Object.keys(data[0]) as column}
							<th on:click={() => handleSort(column)}>
								<div class="th-content">
									{column}
									{#if sortColumn === column}
										<span class="sort-indicator">
											{#if sortDirection === 'asc'}↑{:else}↓{/if}
										</span>
									{/if}
								</div>
							</th>
						{/each}
					{/if}
				</tr>
			</thead>
			<tbody>
				{#each paginatedData as row}
					<tr>
						{#each Object.values(row) as cell}
							<td>{cell}</td>
						{/each}
					</tr>
				{/each}
			</tbody>
		</table>
	</div>

	{#if totalPages > 1}
		<div class="pagination">
			<span class="pagination-info">
				Showing {(currentPage - 1) * pageSize + 1} to {Math.min(
					currentPage * pageSize,
					sortedData.length
				)} of {sortedData.length} entries
			</span>
			<div class="pagination-buttons">
				<button
					class="pagination-btn"
					disabled={currentPage === 1}
					on:click={() => goToPage(currentPage - 1)}
				>
					Previous
				</button>
				{#each Array(totalPages) as _, i}
					{#if i + 1 === currentPage || i + 1 === 1 || i + 1 === totalPages || (i + 1 >= currentPage - 1 && i + 1 <= currentPage + 1)}
						<button
							class="pagination-btn"
							class:active={currentPage === i + 1}
							on:click={() => goToPage(i + 1)}
						>
							{i + 1}
						</button>
					{:else if i + 1 === currentPage - 2 || i + 1 === currentPage + 2}
						<span class="pagination-ellipsis">...</span>
					{/if}
				{/each}
				<button
					class="pagination-btn"
					disabled={currentPage === totalPages}
					on:click={() => goToPage(currentPage + 1)}
				>
					Next
				</button>
			</div>
		</div>
	{/if}
</div>

<style>
	.table-container {
		width: 100%;
		background-color: white;
		border-radius: 0.75rem;
		box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
		padding: 1.5rem;
		margin-top: 2rem;
	}

	.table-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 1rem;
		flex-wrap: wrap;
		gap: 1rem;
	}

	.search-box {
		position: relative;
		width: 100%;
		max-width: 16rem;
	}

	.search-input {
		width: 100%;
		padding: 0.5rem 1rem;
		border: 1px solid #d1d5db;
		border-radius: 0.5rem;
		font-size: 0.875rem;
	}

	.search-input:focus {
		outline: none;
		border-color: #3b82f6;
		box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
	}

	.page-size-selector {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		font-size: 0.875rem;
		color: #4b5563;
	}

	.select-input {
		border: 1px solid #d1d5db;
		border-radius: 0.5rem;
		padding: 0.5rem 2rem 0.5rem 0.75rem;
		font-size: 0.875rem;
		background-color: white;
		background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='%236b7280'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M19 9l-7 7-7-7'%3E%3C/path%3E%3C/svg%3E");
		background-repeat: no-repeat;
		background-position: right 0.5rem center;
		background-size: 1.25rem;
		appearance: none;
		cursor: pointer;
		transition: all 0.2s ease;
		color: #374151;
		font-weight: 500;
	}

	.select-input:hover {
		border-color: #9ca3af;
		background-color: #f9fafb;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
	}

	.select-input:focus {
		outline: none;
		border-color: #3b82f6;
		box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
		background-color: white;
	}

	.select-input option {
		cursor: pointer;
		padding: 0.5rem;
	}

	.table-wrapper {
		overflow-x: auto;
	}

	.data-table {
		width: 100%;
		border-collapse: collapse;
	}

	.data-table thead {
		background-color: #f9fafb;
	}

	.data-table th {
		padding: 0.75rem 1.5rem;
		text-align: left;
		font-size: 0.75rem;
		font-weight: 500;
		color: #6b7280;
		text-transform: uppercase;
		letter-spacing: 0.05em;
		cursor: pointer;
		user-select: none;
	}

	.data-table th:hover {
		background-color: #f3f4f6;
	}

	.th-content {
		display: flex;
		align-items: center;
		gap: 0.25rem;
	}

	.sort-indicator {
		color: #3b82f6;
	}

	.data-table tbody tr {
		border-top: 1px solid #e5e7eb;
	}

	.data-table tbody tr:hover {
		background-color: #f9fafb;
	}

	.data-table td {
		padding: 1rem 1.5rem;
		white-space: nowrap;
		font-size: 0.875rem;
		color: #111827;
	}

	.pagination {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-top: 1rem;
		flex-wrap: wrap;
		gap: 1rem;
	}

	.pagination-info {
		font-size: 0.875rem;
		color: #6b7280;
	}

	.pagination-buttons {
		display: flex;
		gap: 0.5rem;
	}

	.pagination-btn {
		padding: 0.25rem 0.75rem;
		border-radius: 0.375rem;
		border: 1px solid #d1d5db;
		background-color: white;
		color: #374151;
		font-size: 0.875rem;
		cursor: pointer;
		transition: background-color 0.2s;
	}

	.pagination-btn:hover:not(:disabled) {
		background-color: #f3f4f6;
	}

	.pagination-btn:disabled {
		opacity: 0.5;
		cursor: not-allowed;
	}

	.pagination-btn.active {
		background-color: #2563eb;
		color: white;
		border-color: #2563eb;
	}

	.pagination-ellipsis {
		padding: 0 0.5rem;
		color: #6b7280;
	}
</style>
