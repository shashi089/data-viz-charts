<script lang="ts">
	import { createEventDispatcher } from 'svelte';
	import Papa from 'papaparse';
	import * as XLSX from 'xlsx';

	const dispatch = createEventDispatcher();
	let isDragging = false;
	let fileName = '';
	let error = '';
	let loading = false;
	let successMessage = '';

	function handleDragOver(e: DragEvent) {
		e.preventDefault();
		isDragging = true;
	}

	function handleDragLeave() {
		isDragging = false;
	}

	function handleDrop(e: DragEvent) {
		e.preventDefault();
		isDragging = false;
		const files = e.dataTransfer?.files;
		if (files && files.length > 0) {
			processFile(files[0]);
		}
	}

	function handleFileSelect(e: Event) {
		const target = e.target as HTMLInputElement;
		if (target.files && target.files.length > 0) {
			processFile(target.files[0]);
		}
	}

	async function processFile(file: File) {
		fileName = file.name;
		error = '';
		successMessage = '';
		loading = true;

		try {
			if (file.type === 'text/csv' || file.name.endsWith('.csv')) {
				parseCSV(file);
			} else if (
				file.type === 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' ||
				file.name.endsWith('.xlsx') ||
				file.name.endsWith('.xls')
			) {
				parseExcel(file);
			} else {
				error = 'Unsupported file format. Please upload CSV or Excel files.';
				loading = false;
			}
		} catch (err) {
			error = 'Error processing file.';
			console.error(err);
			loading = false;
		}
	}

	function parseCSV(file: File) {
		Papa.parse(file, {
			header: true,
			skipEmptyLines: true,
			complete: (results) => {
				loading = false;
				if (results.errors.length > 0) {
					error = 'Error parsing CSV: ' + results.errors[0].message;
				} else {
					successMessage = 'File uploaded successfully!';
					dispatch('dataLoaded', results.data);
				}
			},
			error: (err) => {
				loading = false;
				error = 'Error parsing CSV: ' + err.message;
			}
		});
	}

	function parseExcel(file: File) {
		const reader = new FileReader();
		reader.onload = (e) => {
			try {
				const data = new Uint8Array(e.target?.result as ArrayBuffer);
				const workbook = XLSX.read(data, { type: 'array' });
				const firstSheetName = workbook.SheetNames[0];
				const worksheet = workbook.Sheets[firstSheetName];
				const jsonData = XLSX.utils.sheet_to_json(worksheet);
				loading = false;
				successMessage = 'File uploaded successfully!';
				dispatch('dataLoaded', jsonData);
			} catch (err) {
				loading = false;
				error = 'Error parsing Excel file.';
			}
		};
		reader.onerror = () => {
			loading = false;
			error = 'Error reading file.';
		};
		reader.readAsArrayBuffer(file);
	}

	function resetUpload() {
		fileName = '';
		error = '';
		successMessage = '';
		loading = false;
		dispatch('reset');
	}
</script>

<div
	class="upload-container"
	class:dragging={isDragging}
	on:dragover={handleDragOver}
	on:dragleave={handleDragLeave}
	on:drop={handleDrop}
	role="region"
	aria-label="File Upload Drop Zone"
>
	<div style="text-align: center;">
		<svg
			style="margin: 0 auto; height: 48px; width: 48px; color: #9ca3af;"
			stroke="currentColor"
			fill="none"
			viewBox="0 0 48 48"
			aria-hidden="true"
		>
			<path
				d="M28 8H12a4 4 0 00-4 4v20m32-12v8m0 0v8a4 4 0 01-4 4H12a4 4 0 01-4-4v-4m32-4l-3.172-3.172a4 4 0 00-5.656 0L28 28M8 32l9.172-9.172a4 4 0 015.656 0L28 28m0 0l4 4m4-24h8m-4-4v8m-12 4h.02"
				stroke-width="2"
				stroke-linecap="round"
				stroke-linejoin="round"
			/>
		</svg>
		<h3 style="margin-top: 0.5rem; font-size: 0.875rem; font-weight: 500; color: #111827;">
			Upload a file
		</h3>
		<p style="margin-top: 0.25rem; font-size: 0.75rem; color: #6b7280;">
			CSV, XLS, XLSX up to 10MB
		</p>

		<div style="margin-top: 1.5rem;">
			<label for="file-upload" class="file-button" class:disabled={successMessage !== ''}>
				<span>Select File</span>
				<input
					id="file-upload"
					name="file-upload"
					type="file"
					style="position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px; overflow: hidden; clip: rect(0,0,0,0); border: 0;"
					accept=".csv, .xls, .xlsx"
					on:change={handleFileSelect}
					disabled={successMessage !== ''}
				/>
			</label>
		</div>

		{#if fileName}
			<p style="margin-top: 1rem; font-size: 0.875rem; color: #4b5563;">
				Selected: <span style="font-weight: 500;">{fileName}</span>
			</p>
		{/if}

		{#if loading}
			<div class="loading-container">
				<div class="spinner"></div>
				<p style="margin-top: 0.5rem; font-size: 0.875rem; color: #3b82f6; font-weight: 500;">
					Uploading and processing file...
				</p>
			</div>
		{/if}

		{#if successMessage}
			<div class="success-container">
				<svg class="success-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"
					/>
				</svg>
				<p style="margin: 0; font-size: 0.875rem; color: #10b981; font-weight: 500;">
					{successMessage}
				</p>
			</div>
			<div style="margin-top: 1rem;">
				<button class="reset-button" on:click={resetUpload}>
					<svg class="reset-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"
						/>
					</svg>
					Reset & Upload New File
				</button>
			</div>
		{/if}

		{#if error}
			<div class="error-container">
				<svg class="error-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
					/>
				</svg>
				<p style="margin: 0; font-size: 0.875rem; color: #ef4444; font-weight: 500;">{error}</p>
			</div>
		{/if}
	</div>
</div>

<style>
	.upload-container {
		width: 100%;
		max-width: 42rem;
		margin: 0 auto;
		padding: 2rem;
		background-color: white;
		border-radius: 0.75rem;
		box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
		border: 2px dashed #d1d5db;
		transition: all 0.3s ease-in-out;
	}

	.upload-container.dragging {
		border-color: #3b82f6;
		background-color: #eff6ff;
	}

	.file-button {
		position: relative;
		cursor: pointer;
		border-radius: 0.375rem;
		background-color: #2563eb;
		padding: 0.625rem 1rem;
		font-size: 0.875rem;
		font-weight: 500;
		color: white;
		display: inline-block;
		transition: background-color 0.2s;
	}

	.file-button:hover {
		background-color: #1d4ed8;
	}

	.file-button.disabled {
		opacity: 0.5;
		cursor: not-allowed;
		background-color: #9ca3af;
	}

	.file-button.disabled:hover {
		background-color: #9ca3af;
		transform: none;
	}

	.loading-container {
		margin-top: 1rem;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.5rem;
	}

	.spinner {
		width: 32px;
		height: 32px;
		border: 3px solid #e5e7eb;
		border-top-color: #3b82f6;
		border-radius: 50%;
		animation: spin 0.8s linear infinite;
	}

	@keyframes spin {
		to {
			transform: rotate(360deg);
		}
	}

	.success-container {
		margin-top: 1rem;
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 0.5rem;
		padding: 0.75rem;
		background-color: #d1fae5;
		border-radius: 0.5rem;
		animation: slideIn 0.3s ease-out;
	}

	.success-icon {
		width: 20px;
		height: 20px;
		color: #10b981;
	}

	.error-container {
		margin-top: 1rem;
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 0.5rem;
		padding: 0.75rem;
		background-color: #fee2e2;
		border-radius: 0.5rem;
		animation: slideIn 0.3s ease-out;
	}

	.error-icon {
		width: 20px;
		height: 20px;
		color: #ef4444;
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

	.reset-button {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0.625rem 1.25rem;
		background-color: #6366f1;
		color: white;
		border: none;
		border-radius: 0.5rem;
		font-size: 0.875rem;
		font-weight: 500;
		cursor: pointer;
		transition: all 0.2s ease;
		box-shadow: 0 2px 4px rgba(99, 102, 241, 0.2);
	}

	.reset-button:hover {
		background-color: #4f46e5;
		transform: translateY(-1px);
		box-shadow: 0 4px 8px rgba(99, 102, 241, 0.3);
	}

	.reset-button:active {
		transform: translateY(0);
	}

	.reset-icon {
		width: 18px;
		height: 18px;
	}
</style>
