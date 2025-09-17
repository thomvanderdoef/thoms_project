<script lang="ts">
	import { onMount } from 'svelte';
	import { page } from '$app/stores';

	let activityType = '';
	let lessonFocus = '';
	let classData: any[] = [];
	let sortColumn = '';
	let sortDirection = 'asc';

	onMount(() => {
		// Get activity info from URL params
		const urlParams = new URLSearchParams(window.location.search);
		activityType = urlParams.get('activity') || 'Probe';
		lessonFocus = urlParams.get('lesson') || 'v/v/';

		// Generate fake class data
		generateClassData();
	});

	function generateClassData() {
		const students = [
			'Anderson, Emma', 'Baker, Liam', 'Clark, Sophia', 'Davis, Noah', 'Evans, Olivia',
			'Fisher, Ethan', 'Garcia, Ava', 'Harris, Mason', 'Johnson, Isabella', 'Kim, Lucas',
			'Lopez, Mia', 'Miller, Jackson', 'Nelson, Charlotte', 'Parker, Aiden', 'Quinn, Amelia',
			'Roberts, Owen', 'Smith, Harper', 'Taylor, Elijah', 'Turner, Ella', 'Williams, Logan',
			'Young, Grace', 'Zhang, Carter', 'Brown, Lily', 'Wilson, Caleb'
		];

		classData = students.map(name => {
			const baseAccuracy = Math.random() * 0.5 + 0.4; // 40-90% accuracy
			const performance = getPerformanceLevel(baseAccuracy);

			switch (activityType) {
				case 'Probe':
					return generateProbeData(name, baseAccuracy, performance);
				case 'Diagnostic':
					return generateDiagnosticData(name, baseAccuracy, performance);
				case 'Practice':
					return generatePracticeData(name, baseAccuracy, performance);
				case 'Assignment':
					return generateAssignmentData(name, baseAccuracy, performance);
				default:
					return generateProbeData(name, baseAccuracy, performance);
			}
		});
	}

	function getPerformanceLevel(accuracy: number) {
		if (accuracy >= 0.85) return 'Core';
		if (accuracy >= 0.75) return 'Strategic';
		return 'Intensive';
	}

	function generateProbeData(name: string, baseAccuracy: number, performance: string) {
		const totalItems = 10;
		const itemsCorrect = Math.round(baseAccuracy * totalItems);
		const accuracy = Math.round((itemsCorrect / totalItems) * 100);

		return {
			name,
			accuracy,
			itemsCorrect,
			totalItems,
			wcpm: Math.round(Math.random() * 30 + 30), // 30-60 WCPM
			latency: Math.round(Math.random() * 1000 + 1000), // 1-2 seconds
			status: performance,
			recommendation: performance === 'Intensive' ? 'Additional practice needed' :
						   performance === 'Strategic' ? 'Monitor progress' : 'Continue instruction'
		};
	}

	function generateDiagnosticData(name: string, baseAccuracy: number, performance: string) {
		const errorPatterns = ['Letter reversal', 'Vowel confusion', 'Blending difficulty', 'Final consonant'];
		const strengthAreas = ['Initial sounds', 'CVC words', 'Phoneme awareness', 'Letter recognition'];

		return {
			name,
			accuracy: Math.round(baseAccuracy * 100),
			errorPatterns: Math.floor(Math.random() * 3) + 1,
			primaryError: errorPatterns[Math.floor(Math.random() * errorPatterns.length)],
			strengths: strengthAreas[Math.floor(Math.random() * strengthAreas.length)],
			status: performance,
			intervention: performance === 'Intensive' ? 'Tier 3 support' :
						 performance === 'Strategic' ? 'Tier 2 support' : 'Core instruction'
		};
	}

	function generatePracticeData(name: string, baseAccuracy: number, performance: string) {
		return {
			name,
			trials: Math.floor(Math.random() * 10) + 10, // 10-20 trials
			accuracy: Math.round(baseAccuracy * 100),
			improvement: `+${Math.floor(Math.random() * 20)}%`,
			timeSpent: `${Math.floor(Math.random() * 10) + 5} min`,
			status: performance,
			nextSteps: performance === 'Intensive' ? 'Continue practice' :
					  performance === 'Strategic' ? 'Advance to next skill' : 'Mastery achieved'
		};
	}

	function generateAssignmentData(name: string, baseAccuracy: number, performance: string) {
		const totalActivities = 5;
		const completed = Math.floor(Math.random() * 2) + 3; // 3-5 activities

		return {
			name,
			activitiesComplete: completed,
			totalActivities,
			overallAccuracy: Math.round(baseAccuracy * 100),
			timeSpent: `${Math.floor(Math.random() * 15) + 10} min`,
			status: performance,
			progress: completed === totalActivities ? 'Complete' : 'In Progress'
		};
	}

	function sortTable(column: string) {
		if (sortColumn === column) {
			sortDirection = sortDirection === 'asc' ? 'desc' : 'asc';
		} else {
			sortColumn = column;
			sortDirection = 'asc';
		}

		classData = classData.sort((a, b) => {
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
	}

	function getStatusColor(status: string) {
		switch (status) {
			case 'Core': return 'table-success';
			case 'Strategic': return 'table-warning';
			case 'Intensive': return 'table-danger';
			default: return '';
		}
	}

	function calculateClassAverage(field: string) {
		const values = classData.map(student => student[field]).filter(val => typeof val === 'number');
		return values.length > 0 ? Math.round(values.reduce((sum, val) => sum + val, 0) / values.length) : 0;
	}

	function getStatusCounts() {
		const counts = { Core: 0, Strategic: 0, Intensive: 0 };
		classData.forEach(student => {
			counts[student.status as keyof typeof counts]++;
		});
		return counts;
	}
</script>

<svelte:head>
	<title>Class Results - {activityType}: {lessonFocus}</title>
</svelte:head>

<div class="container-fluid py-4">
	<div class="row mb-4">
		<div class="col-12">
			<div class="d-flex justify-content-between align-items-center mb-3">
				<div>
					<h1 class="h3 mb-1">Class Results Dashboard</h1>
					<p class="text-muted mb-0">
						{activityType} Assessment - Lesson Focus: {lessonFocus}
					</p>
				</div>
				<div>
					<button class="btn btn-outline-secondary" on:click={() => window.close()}>
						Close
					</button>
				</div>
			</div>

			<!-- Class Summary Cards -->
			<div class="row mb-4">
				<div class="col-md-3">
					<div class="card border-success">
						<div class="card-body text-center">
							<h5 class="card-title text-success">Core</h5>
							<h2 class="display-6">{getStatusCounts().Core}</h2>
							<small class="text-muted">students</small>
						</div>
					</div>
				</div>
				<div class="col-md-3">
					<div class="card border-warning">
						<div class="card-body text-center">
							<h5 class="card-title text-warning">Strategic</h5>
							<h2 class="display-6">{getStatusCounts().Strategic}</h2>
							<small class="text-muted">students</small>
						</div>
					</div>
				</div>
				<div class="col-md-3">
					<div class="card border-danger">
						<div class="card-body text-center">
							<h5 class="card-title text-danger">Intensive</h5>
							<h2 class="display-6">{getStatusCounts().Intensive}</h2>
							<small class="text-muted">students</small>
						</div>
					</div>
				</div>
				<div class="col-md-3">
					<div class="card border-primary">
						<div class="card-body text-center">
							<h5 class="card-title text-primary">Class Avg</h5>
							<h2 class="display-6">{calculateClassAverage('accuracy')}%</h2>
							<small class="text-muted">accuracy</small>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>

	<div class="row">
		<div class="col-12">
			<div class="card">
				<div class="card-body">
					<div class="table-responsive">
						<table class="table table-hover">
							<thead class="table-dark">
								<tr>
									<th scope="col" class="sortable" on:click={() => sortTable('name')}>
										Student Name
										{#if sortColumn === 'name'}
											<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
										{/if}
									</th>

									{#if activityType === 'Probe'}
										<th scope="col" class="sortable" on:click={() => sortTable('accuracy')}>
											Accuracy
											{#if sortColumn === 'accuracy'}
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
										<th scope="col" class="sortable" on:click={() => sortTable('latency')}>
											Avg Latency (ms)
											{#if sortColumn === 'latency'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
									{:else if activityType === 'Diagnostic'}
										<th scope="col" class="sortable" on:click={() => sortTable('accuracy')}>
											Accuracy
											{#if sortColumn === 'accuracy'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
										<th scope="col" class="sortable" on:click={() => sortTable('errorPatterns')}>
											Error Patterns
											{#if sortColumn === 'errorPatterns'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
										<th scope="col">Primary Error</th>
										<th scope="col">Strengths</th>
									{:else if activityType === 'Practice'}
										<th scope="col" class="sortable" on:click={() => sortTable('trials')}>
											Trials
											{#if sortColumn === 'trials'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
										<th scope="col" class="sortable" on:click={() => sortTable('accuracy')}>
											Accuracy
											{#if sortColumn === 'accuracy'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
										<th scope="col">Improvement</th>
										<th scope="col">Time Spent</th>
									{:else if activityType === 'Assignment'}
										<th scope="col" class="sortable" on:click={() => sortTable('activitiesComplete')}>
											Activities Complete
											{#if sortColumn === 'activitiesComplete'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
										<th scope="col" class="sortable" on:click={() => sortTable('overallAccuracy')}>
											Overall Accuracy
											{#if sortColumn === 'overallAccuracy'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
										<th scope="col">Time Spent</th>
										<th scope="col">Progress</th>
									{/if}

									<th scope="col" class="sortable" on:click={() => sortTable('status')}>
										Status
										{#if sortColumn === 'status'}
											<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
										{/if}
									</th>
									<th scope="col">Recommendation</th>
								</tr>
							</thead>
							<tbody>
								{#each classData as student}
									<tr class={getStatusColor(student.status)}>
										<td><strong>{student.name}</strong></td>

										{#if activityType === 'Probe'}
											<td>{student.accuracy}%</td>
											<td>{student.itemsCorrect}/{student.totalItems}</td>
											<td>{student.wcpm}</td>
											<td>{student.latency}</td>
										{:else if activityType === 'Diagnostic'}
											<td>{student.accuracy}%</td>
											<td>{student.errorPatterns}</td>
											<td>{student.primaryError}</td>
											<td>{student.strengths}</td>
										{:else if activityType === 'Practice'}
											<td>{student.trials}</td>
											<td>{student.accuracy}%</td>
											<td>{student.improvement}</td>
											<td>{student.timeSpent}</td>
										{:else if activityType === 'Assignment'}
											<td>{student.activitiesComplete}/{student.totalActivities}</td>
											<td>{student.overallAccuracy}%</td>
											<td>{student.timeSpent}</td>
											<td>{student.progress}</td>
										{/if}

										<td>
											<span class="badge bg-{student.status === 'Core' ? 'success' : student.status === 'Strategic' ? 'warning' : 'danger'}">
												{student.status}
											</span>
										</td>
										<td class="text-muted">
											{student.recommendation || student.intervention || student.nextSteps}
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
</div>

<style>
	.sortable {
		cursor: pointer;
		user-select: none;
	}

	.sortable:hover {
		background-color: rgba(255, 255, 255, 0.1);
	}

	.table-responsive {
		max-height: 600px;
		overflow-y: auto;
	}

	.card {
		border: none;
		box-shadow: 0 2px 8px rgba(0,0,0,0.1);
	}
</style>