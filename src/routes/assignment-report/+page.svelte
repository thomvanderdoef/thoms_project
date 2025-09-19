<script lang="ts">
	import { onMount } from 'svelte';
	import classResultsData from '$lib/data/class_results.json';
	import suggestedGroupsData from '$lib/data/suggested_groups.json';
	import adminRollupData from '$lib/data/admin_rollup.json';
	import classResultsCsv from '$lib/data/class_results.csv?raw';

	let activityType = '';
	let lessonFocus = '';
	let currentView = 'teacher'; // 'teacher' or 'admin'
	let sortColumn = '';
	let sortDirection = 'asc';

	// Use real data from JSON files
	let classResults = classResultsData;
	let suggestedGroups = suggestedGroupsData;
	let adminRollup = adminRollupData;

	onMount(() => {
		// Get assignment info from URL params
		const urlParams = new URLSearchParams(window.location.search);
		activityType = urlParams.get('activity') || 'Probe';
		lessonFocus = urlParams.get('lesson') || 'v/v/';
	});

	function getRiskColor(risk: string) {
		switch (risk) {
			case 'Core': return 'success';
			case 'Strategic': return 'warning';
			case 'Intensive': return 'danger';
			default: return 'secondary';
		}
	}

	function downloadCSV() {
		const blob = new Blob([classResultsCsv], { type: 'text/csv' });
		const url = window.URL.createObjectURL(blob);
		const a = document.createElement('a');
		a.style.display = 'none';
		a.href = url;
		a.download = `class_results_${new Date().toISOString().split('T')[0]}.csv`;
		document.body.appendChild(a);
		a.click();
		window.URL.revokeObjectURL(url);
		document.body.removeChild(a);
	}

	function assignPractice() {
		alert('Practice activities would be assigned to flagged students');
	}

	function scheduleNextProbe() {
		alert('Next probe would be scheduled for the class');
	}

	function sortTable(column: string) {
		if (sortColumn === column) {
			sortDirection = sortDirection === 'asc' ? 'desc' : 'asc';
		} else {
			sortColumn = column;
			sortDirection = 'asc';
		}

		// Sort the students array
		classResults.students = classResults.students.sort((a, b) => {
			let aVal = a[column];
			let bVal = b[column];

			// Handle numeric sorting
			if (typeof aVal === 'number' && typeof bVal === 'number') {
				return sortDirection === 'asc' ? aVal - bVal : bVal - aVal;
			}

			// Handle string sorting
			aVal = String(aVal).toLowerCase();
			bVal = String(bVal).toLowerCase();

			if (sortDirection === 'asc') {
				return aVal < bVal ? -1 : aVal > bVal ? 1 : 0;
			} else {
				return aVal > bVal ? -1 : aVal < bVal ? 1 : 0;
			}
		});

		// Trigger reactivity
		classResults = classResults;
	}

	// Flag logic to connect data points with suggested groups
	function getStudentGroupFlags(studentName: string) {
		// Find which group this student belongs to
		const group = suggestedGroups.find(g => g.students.includes(studentName));
		return group ? group.groupId : null;
	}

	function shouldFlagAccuracy(student: any) {
		// Flag accuracy if student is in a group and accuracy is below threshold
		return getStudentGroupFlags(student.name) && student.accuracyPct < 70;
	}

	function shouldFlagWCPM(student: any) {
		// Flag WCPM for fluency group (Group C) students with slow rates
		const groupId = getStudentGroupFlags(student.name);
		return groupId === 'grp_C' && (student.wcpm < 50 || student.miscues.some(m => m.includes('slow rate')));
	}

	function shouldFlagMiscues(student: any, miscue: string) {
		const groupId = getStudentGroupFlags(student.name);

		// Group A (v/b contrast) - flag v/b confusion errors
		if (groupId === 'grp_A' && miscue.includes('/v/→/b/')) {
			return true;
		}

		// Group B (short a review) - flag short a errors
		if (groupId === 'grp_B' && miscue.includes('short a→long a')) {
			return true;
		}

		// Group C (fluency) - flag rate issues
		if (groupId === 'grp_C' && miscue.includes('slow rate')) {
			return true;
		}

		return false;
	}
</script>

<svelte:head>
	<title>{activityType} Assessment - Lesson: {lessonFocus}</title>
</svelte:head>

<div class="container-fluid py-4">
	<!-- Header with Tab Navigation -->
	<div class="row mb-4">
		<div class="col-12">
			<div class="d-flex justify-content-between align-items-center mb-3">
				<div>
					<h1 class="h3 mb-0">{classResults.lesson}</h1>
					<p class="text-muted mb-0">{classResults.students.length} Students • {classResults.activityType}</p>
				</div>
				<div>
					<button class="btn btn-outline-secondary me-2" on:click={() => window.print()}>
						Print Report
					</button>
					<button class="btn btn-outline-secondary" on:click={() => window.close()}>
						Close
					</button>
				</div>
			</div>

			<!-- Tab Navigation -->
			<ul class="nav nav-tabs" id="viewTabs" role="tablist">
				<li class="nav-item" role="presentation">
					<button
						class="nav-link {currentView === 'teacher' ? 'active' : ''}"
						on:click={() => currentView = 'teacher'}
						type="button"
					>
						Teacher View
					</button>
				</li>
				<li class="nav-item" role="presentation">
					<button
						class="nav-link {currentView === 'admin' ? 'active' : ''}"
						on:click={() => currentView = 'admin'}
						type="button"
					>
						Admin View
					</button>
				</li>
			</ul>
		</div>
	</div>

	<!-- Teacher View -->
	{#if currentView === 'teacher'}
		<!-- Performance Metrics Header -->
		<div class="row mb-4">
			<div class="col-md-3">
				<div class="card border-success">
					<div class="card-body text-center">
						<h5 class="card-title text-success">Core</h5>
						<h2 class="display-6">{classResults.metrics.coreCount}</h2>
						<small class="text-muted">students</small>
					</div>
				</div>
			</div>
			<div class="col-md-3">
				<div class="card border-warning">
					<div class="card-body text-center">
						<h5 class="card-title text-warning">Strategic</h5>
						<h2 class="display-6">{classResults.metrics.strategicCount}</h2>
						<small class="text-muted">students</small>
					</div>
				</div>
			</div>
			<div class="col-md-3">
				<div class="card border-danger">
					<div class="card-body text-center">
						<h5 class="card-title text-danger">Intensive</h5>
						<h2 class="display-6">{classResults.metrics.intensiveCount}</h2>
						<small class="text-muted">students</small>
					</div>
				</div>
			</div>
			<div class="col-md-3">
				<div class="card border-primary">
					<div class="card-body text-center">
						<h5 class="card-title text-primary">Avg Accuracy</h5>
						<h2 class="display-6">{classResults.metrics.avgAccuracyPct}%</h2>
						<small class="text-muted">class average</small>
					</div>
				</div>
			</div>
		</div>

		<!-- Individual Student Results -->
		<div class="row mb-4">
			<div class="col-12">
				<h4 class="mb-3">Individual Student Results</h4>
				<div class="card">
					<div class="card-body">
						<div class="table-responsive">
							<table class="table">
								<thead class="table-dark">
									<tr>
										<th scope="col" class="sortable" on:click={() => sortTable('name')}>
											Student
											{#if sortColumn === 'name'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
										<th scope="col" class="sortable" on:click={() => sortTable('accuracyPct')}>
											Accuracy
											{#if sortColumn === 'accuracyPct'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
										<th scope="col" class="sortable" on:click={() => sortTable('itemsCorrect')}>
											Items Correct
											{#if sortColumn === 'itemsCorrect'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
										<th scope="col" class="sortable" on:click={() => sortTable('wcpm')}>
											WCPM
											{#if sortColumn === 'wcpm'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
										<th scope="col" class="sortable" on:click={() => sortTable('risk')}>
											Risk
											{#if sortColumn === 'risk'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
										<th scope="col">Miscues</th>
									</tr>
								</thead>
								<tbody>
									{#each classResults.students as student}
										<tr class="table-{getRiskColor(student.risk)}">
											<td>
												<strong>{student.name}</strong>
											</td>
											<td>
												{student.accuracyPct}%
												{#if shouldFlagAccuracy(student)}
													<span class="ms-1">🚩</span>
												{/if}
											</td>
											<td>{student.itemsCorrect}/{student.itemsTotal}</td>
											<td>
												{student.wcpm}
												{#if shouldFlagWCPM(student)}
													<span class="ms-1">🚩</span>
												{/if}
											</td>
											<td>
												<span class="badge bg-{getRiskColor(student.risk)}">
													{student.risk}
												</span>
											</td>
											<td>
												{#if student.miscues.length > 0}
													<div class="d-flex flex-column gap-1">
														{#each student.miscues as miscue}
															<small class="text-muted d-flex align-items-center gap-1">
																<span>{miscue}</span>
																{#if shouldFlagMiscues(student, miscue)}
																	<span>🚩</span>
																{/if}
															</small>
														{/each}
													</div>
												{:else}
													<small class="text-success">No errors</small>
												{/if}
											</td>
										</tr>
									{/each}
								</tbody>
							</table>
						</div>
					</div>
				</div>
			</div>
		</div>

		<!-- Suggested Groups Panel -->
		<div class="row mb-4">
			<div class="col-12">
				<h4 class="mb-3">Suggested Groups</h4>
				<div class="row g-3">
					{#each suggestedGroups as group}
						<div class="col-md-4">
							<div class="card h-100">
								<div class="card-body">
									<h6 class="card-title text-primary">🚩 {group.name}</h6>
									<p class="text-muted small mb-2">{group.targetSkill}</p>
									<div class="mb-3">
										<strong>Students:</strong>
										<ul class="list-unstyled mb-0 ms-2">
											{#each group.students as studentName}
												<li>• {studentName}</li>
											{/each}
										</ul>
									</div>
									<div class="alert alert-info alert-sm">
										<strong>Next Step:</strong> {group.recommendedNext}
									</div>
								</div>
							</div>
						</div>
					{/each}
				</div>
			</div>
		</div>

		<!-- Next Steps Actions -->
		<div class="row">
			<div class="col-12">
				<h4 class="mb-3">Next Steps</h4>
				<div class="d-flex gap-3">
					<button class="btn btn-primary" on:click={assignPractice}>
						Assign Practice
					</button>
					<button class="btn btn-outline-primary" on:click={scheduleNextProbe}>
						Schedule Next Probe
					</button>
				</div>
			</div>
		</div>
	{/if}

	<!-- Admin View -->
	{#if currentView === 'admin'}
		<div class="row mb-4">
			<div class="col-md-8">
				<!-- Performance Distribution Chart -->
				<div class="card mb-4">
					<div class="card-body">
						<h5 class="card-title">Performance Distribution</h5>
						<div class="row text-center">
							<div class="col-md-4">
								<div class="progress mb-2" style="height: 30px;">
									<div class="progress-bar bg-success" role="progressbar"
										style="width: {(adminRollup.rollup.core / (adminRollup.rollup.core + adminRollup.rollup.strategic + adminRollup.rollup.intensive)) * 100}%">
										{adminRollup.rollup.core} Core
									</div>
								</div>
							</div>
							<div class="col-md-4">
								<div class="progress mb-2" style="height: 30px;">
									<div class="progress-bar bg-warning" role="progressbar"
										style="width: {(adminRollup.rollup.strategic / (adminRollup.rollup.core + adminRollup.rollup.strategic + adminRollup.rollup.intensive)) * 100}%">
										{adminRollup.rollup.strategic} Strategic
									</div>
								</div>
							</div>
							<div class="col-md-4">
								<div class="progress mb-2" style="height: 30px;">
									<div class="progress-bar bg-danger" role="progressbar"
										style="width: {(adminRollup.rollup.intensive / (adminRollup.rollup.core + adminRollup.rollup.strategic + adminRollup.rollup.intensive)) * 100}%">
										{adminRollup.rollup.intensive} Intensive
									</div>
								</div>
							</div>
						</div>
					</div>
				</div>

				<!-- Progress Trendline -->
				<div class="card">
					<div class="card-body">
						<h5 class="card-title">Class Progress Over Time</h5>
						<div class="row align-items-end" style="height: 200px;">
							{#each adminRollup.trendline as point, index}
								<div class="col text-center">
									<div class="bg-primary rounded" style="height: {(point.avgAccuracyPct / 100) * 150}px; width: 100%; margin-bottom: 10px;"></div>
									<small class="text-muted">{point.label}</small>
									<br>
									<strong>{point.avgAccuracyPct}%</strong>
								</div>
							{/each}
						</div>
					</div>
				</div>
			</div>

			<div class="col-md-4">
				<!-- Compliance Badges -->
				<div class="card">
					<div class="card-body">
						<h5 class="card-title">Compliance & Standards</h5>
						<div class="d-flex flex-column gap-2">
							{#each adminRollup.badges as badge}
								<span class="badge bg-success p-2">✓ {badge}</span>
							{/each}
						</div>

						<div class="mt-4">
							<h6>Cohort Information</h6>
							<p class="text-muted mb-2">{adminRollup.cohort}</p>
							<p class="small text-muted">Assessment ID: {adminRollup.assignmentId}</p>
						</div>

						<div class="mt-4">
							<button class="btn btn-outline-primary w-100" on:click={downloadCSV}>
								📊 Download Report
							</button>
						</div>
					</div>
				</div>
			</div>
		</div>
	{/if}
</div>

<style>
	.alert-sm {
		padding: 0.5rem 0.75rem;
		font-size: 0.875rem;
	}

	.sortable {
		cursor: pointer;
		user-select: none;
	}

	.sortable:hover {
		background-color: rgba(255, 255, 255, 0.2) !important;
	}

	.table-dark .sortable:hover {
		background-color: rgba(0, 0, 0, 0.3) !important;
		transform: scale(1.02) !important;
		transition: all 0.2s ease !important;
	}

	.table tbody tr:hover {
		opacity: 0.9;
		transform: scale(1.01);
		transition: all 0.2s ease;
	}

	.nav-tabs .nav-link {
		cursor: pointer;
	}

	@media print {
		.btn, .btn-group, .nav-tabs {
			display: none !important;
		}
	}
</style>