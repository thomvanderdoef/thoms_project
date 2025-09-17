<script lang="ts">
	import { onMount } from 'svelte';

	let activityType = '';
	let lessonFocus = '';
	let assignedStudents: any[] = [];
	let studentResults: any[] = [];
	let sortColumn = '';
	let sortDirection = 'asc';

	// Sample class roster for reference
	const classRoster = [
		{ id: 'std001', name: 'Anderson, Emma', performance: 'Core', avatar: '👧🏼' },
		{ id: 'std002', name: 'Baker, Liam', performance: 'Strategic', avatar: '👦🏻' },
		{ id: 'std003', name: 'Clark, Sophia', performance: 'Core', avatar: '👧🏽' },
		{ id: 'std004', name: 'Davis, Noah', performance: 'Intensive', avatar: '👦🏾' },
		{ id: 'std005', name: 'Evans, Olivia', performance: 'Strategic', avatar: '👧🏻' },
		{ id: 'std006', name: 'Fisher, Ethan', performance: 'Core', avatar: '👦🏼' },
		{ id: 'std007', name: 'Garcia, Ava', performance: 'Intensive', avatar: '👧🏽' },
		{ id: 'std008', name: 'Harris, Mason', performance: 'Core', avatar: '👦🏻' },
		{ id: 'std009', name: 'Johnson, Isabella', performance: 'Strategic', avatar: '👧🏾' },
		{ id: 'std010', name: 'Kim, Lucas', performance: 'Core', avatar: '👦🏻' },
		{ id: 'std011', name: 'Lopez, Mia', performance: 'Intensive', avatar: '👧🏽' },
		{ id: 'std012', name: 'Miller, Jackson', performance: 'Strategic', avatar: '👦🏼' },
		{ id: 'std013', name: 'Nelson, Charlotte', performance: 'Core', avatar: '👧🏻' },
		{ id: 'std014', name: 'Parker, Aiden', performance: 'Intensive', avatar: '👦🏽' },
		{ id: 'std015', name: 'Quinn, Amelia', performance: 'Core', avatar: '👧🏼' },
		{ id: 'std016', name: 'Roberts, Owen', performance: 'Strategic', avatar: '👦🏻' },
		{ id: 'std017', name: 'Smith, Harper', performance: 'Core', avatar: '👧🏽' },
		{ id: 'std018', name: 'Taylor, Elijah', performance: 'Intensive', avatar: '👦🏾' },
		{ id: 'std019', name: 'Turner, Ella', performance: 'Strategic', avatar: '👧🏻' },
		{ id: 'std020', name: 'Williams, Logan', performance: 'Core', avatar: '👦🏼' }
	];

	onMount(() => {
		// Get assignment info from URL params
		const urlParams = new URLSearchParams(window.location.search);
		activityType = urlParams.get('activity') || 'Probe';
		lessonFocus = urlParams.get('lesson') || 'v/v/';
		const studentIds = urlParams.get('students')?.split(',') || [];

		// Get assigned students info
		assignedStudents = classRoster.filter(student => studentIds.includes(student.id));

		// If no students were passed, show a sample of students for demo purposes
		if (assignedStudents.length === 0) {
			assignedStudents = classRoster.slice(0, 8); // Show first 8 students as sample
		}

		// Generate results that match the expected data from preview
		generateStudentResults();
	});

	function generateStudentResults() {
		studentResults = assignedStudents.map(student => {
			// Base accuracy on student's performance level
			let baseAccuracy: number;
			switch (student.performance) {
				case 'Core': baseAccuracy = 0.8 + Math.random() * 0.15; break; // 80-95%
				case 'Strategic': baseAccuracy = 0.65 + Math.random() * 0.2; break; // 65-85%
				case 'Intensive': baseAccuracy = 0.4 + Math.random() * 0.3; break; // 40-70%
				default: baseAccuracy = 0.7;
			}

			// Ensure it matches expected ranges from preview cards
			if (activityType === 'Probe') {
				baseAccuracy = Math.max(0.7, Math.min(0.85, baseAccuracy)); // 70-85% as expected
			} else if (activityType === 'Diagnostic') {
				baseAccuracy = Math.max(0.65, Math.min(0.8, baseAccuracy)); // 65-80% as expected
			}

			switch (activityType) {
				case 'Probe':
					return generateProbeResults(student, baseAccuracy);
				case 'Diagnostic':
					return generateDiagnosticResults(student, baseAccuracy);
				case 'Practice':
					return generatePracticeResults(student, baseAccuracy);
				case 'Assignment':
					return generateAssignmentResults(student, baseAccuracy);
				default:
					return generateProbeResults(student, baseAccuracy);
			}
		});
	}

	function generateProbeResults(student: any, baseAccuracy: number) {
		const totalItems = 12;
		const itemsCorrect = Math.round(baseAccuracy * totalItems);
		const accuracy = Math.round((itemsCorrect / totalItems) * 100);
		const wcpm = Math.round(45 + Math.random() * 15); // 45-60 WCPM as expected
		const errors = Math.max(1, totalItems - itemsCorrect);

		return {
			...student,
			accuracy,
			itemsCorrect,
			totalItems,
			wcpm,
			errors,
			latency: Math.round(Math.random() * 800 + 1200), // 1.2-2.0 seconds
			miscues: generateMiscues(errors),
			status: getStatusFromAccuracy(accuracy),
			recommendation: getRecommendation(accuracy, 'Probe'),
			nextSteps: getNextSteps(accuracy, student.performance)
		};
	}

	function generateDiagnosticResults(student: any, baseAccuracy: number) {
		const accuracy = Math.round(baseAccuracy * 100);
		const errorCount = Math.floor(Math.random() * 3) + 2; // 2-4 error types as expected

		return {
			...student,
			accuracy,
			errorPatterns: errorCount,
			primaryErrors: generateErrorPatterns(errorCount),
			strengths: generateStrengths(student.performance),
			needsWork: generateNeedsWork(student.performance),
			status: getStatusFromAccuracy(accuracy),
			intervention: getInterventionLevel(student.performance),
			nextSteps: getDiagnosticNextSteps(student.performance)
		};
	}

	function generatePracticeResults(student: any, baseAccuracy: number) {
		const trials = Math.floor(Math.random() * 8) + 10; // 10-18 trials
		const accuracy = Math.round(baseAccuracy * 100);
		const improvement = Math.floor(Math.random() * 20) + 10; // +10-30% improvement

		return {
			...student,
			trials,
			accuracy,
			improvement: `+${improvement}%`,
			timeSpent: `${Math.floor(Math.random() * 5) + 8} min`, // 8-12 min as expected
			attempts: Math.floor(trials * 0.8),
			mastery: accuracy > 85 ? 'Achieved' : accuracy > 70 ? 'Approaching' : 'Needs Practice',
			status: getStatusFromAccuracy(accuracy),
			nextSteps: getPracticeNextSteps(accuracy)
		};
	}

	function generateAssignmentResults(student: any, baseAccuracy: number) {
		const totalActivities = 5;
		const completed = Math.floor(Math.random() * 2) + 3; // 3-5 activities
		const accuracy = Math.round(baseAccuracy * 100);

		return {
			...student,
			activitiesCompleted: completed,
			totalActivities,
			completionRate: Math.round((completed / totalActivities) * 100),
			overallAccuracy: accuracy,
			timeSpent: `${Math.floor(Math.random() * 10) + 15} min`, // 15-25 min as expected
			status: getStatusFromAccuracy(accuracy),
			progress: completed === totalActivities ? 'Complete' : 'In Progress',
			nextSteps: getAssignmentNextSteps(accuracy, completed, totalActivities)
		};
	}

	function generateMiscues(errorCount: number) {
		const possibleMiscues = [
			'Letter reversal (b/d)',
			'Short vowel confusion',
			'Final consonant omission',
			'Initial blend difficulty',
			'Silent e pattern error'
		];
		return possibleMiscues.slice(0, Math.min(errorCount, 3));
	}

	function generateErrorPatterns(count: number) {
		const patterns = [
			'Letter reversals (b/d, p/q)',
			'Vowel team confusion',
			'Consonant blend errors',
			'Silent letter omissions',
			'Phoneme substitutions'
		];
		return patterns.slice(0, count);
	}

	function generateStrengths(performance: string) {
		const allStrengths = [
			'Initial consonant sounds',
			'CVC word reading',
			'Letter recognition',
			'Phonemic awareness',
			'Blending skills'
		];
		const count = performance === 'Core' ? 4 : performance === 'Strategic' ? 3 : 2;
		return allStrengths.slice(0, count);
	}

	function generateNeedsWork(performance: string) {
		const allNeeds = [
			'Vowel discrimination',
			'Final consonant blends',
			'Multi-syllabic words',
			'Silent e patterns',
			'R-controlled vowels'
		];
		const count = performance === 'Intensive' ? 3 : performance === 'Strategic' ? 2 : 1;
		return allNeeds.slice(0, count);
	}

	function getStatusFromAccuracy(accuracy: number) {
		if (accuracy >= 80) return 'Core';
		if (accuracy >= 65) return 'Strategic';
		return 'Intensive';
	}

	function getRecommendation(accuracy: number, activityType: string) {
		if (accuracy >= 85) return 'Continue current instruction level';
		if (accuracy >= 70) return 'Monitor progress, consider additional practice';
		return 'Provide intensive intervention and support';
	}

	function getInterventionLevel(performance: string) {
		switch (performance) {
			case 'Core': return 'Core instruction sufficient';
			case 'Strategic': return 'Tier 2 intervention recommended';
			case 'Intensive': return 'Tier 3 intensive support needed';
			default: return 'Monitor progress';
		}
	}

	function getNextSteps(accuracy: number, performance: string) {
		if (accuracy >= 85) return 'Advance to next skill level';
		if (accuracy >= 70) return 'Continue practice with current skill';
		return 'Intensive review and reteaching needed';
	}

	function getDiagnosticNextSteps(performance: string) {
		switch (performance) {
			case 'Core': return 'Monitor with periodic check-ins';
			case 'Strategic': return 'Small group targeted practice';
			case 'Intensive': return 'Individual intervention plan';
			default: return 'Continue assessment';
		}
	}

	function getPracticeNextSteps(accuracy: number) {
		if (accuracy >= 85) return 'Skill mastered, advance to next level';
		if (accuracy >= 70) return 'Continue practice sessions';
		return 'Increase practice frequency and support';
	}

	function getAssignmentNextSteps(accuracy: number, completed: number, total: number) {
		if (completed === total && accuracy >= 80) return 'Assignment mastered, ready for next unit';
		if (completed === total) return 'Completed with room for improvement';
		return 'Continue working through remaining activities';
	}

	function getStatusColor(status: string) {
		switch (status) {
			case 'Core': return 'success';
			case 'Strategic': return 'warning';
			case 'Intensive': return 'danger';
			default: return 'secondary';
		}
	}

	function calculateClassAverage(field: string) {
		const values = studentResults.map(result => result[field]).filter(val => typeof val === 'number');
		return values.length > 0 ? Math.round(values.reduce((sum, val) => sum + val, 0) / values.length) : 0;
	}

	function getStatusCounts() {
		const counts = { Core: 0, Strategic: 0, Intensive: 0 };
		studentResults.forEach(result => {
			counts[result.status as keyof typeof counts]++;
		});
		return counts;
	}

	function sortTable(column: string) {
		if (sortColumn === column) {
			sortDirection = sortDirection === 'asc' ? 'desc' : 'asc';
		} else {
			sortColumn = column;
			sortDirection = 'asc';
		}

		studentResults = studentResults.sort((a, b) => {
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
</script>

<svelte:head>
	<title>Assignment Report - {activityType}: {lessonFocus}</title>
</svelte:head>

<div class="container-fluid py-4">
	<div class="row mb-4">
		<div class="col-12">
			<div class="d-flex justify-content-between align-items-center mb-3">
				<div>
					<h1 class="h3 mb-0">{activityType} Assessment - Lesson: {lessonFocus} - {assignedStudents.length} Students</h1>
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

			<!-- Assignment Summary -->
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
							<h5 class="card-title text-primary">Avg Accuracy</h5>
							<h2 class="display-6">{calculateClassAverage('accuracy')}%</h2>
							<small class="text-muted">class average</small>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>

	<!-- Individual Student Results -->
	<div class="row">
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
									{:else if activityType === 'Extended Assignment'}
										<th scope="col" class="sortable" on:click={() => sortTable('completionRate')}>
											Completion
											{#if sortColumn === 'completionRate'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
										<th scope="col" class="sortable" on:click={() => sortTable('overallAccuracy')}>
											Accuracy
											{#if sortColumn === 'overallAccuracy'}
												<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
											{/if}
										</th>
										<th scope="col">Status</th>
									{/if}
									<th scope="col" class="sortable" on:click={() => sortTable('status')}>
										Status
										{#if sortColumn === 'status'}
											<i class="bi bi-arrow-{sortDirection === 'asc' ? 'up' : 'down'}"></i>
										{/if}
									</th>
								</tr>
							</thead>
							<tbody>
								{#each studentResults as result}
									<tr class="table-{getStatusColor(result.status)}">
										<td>
											<div class="d-flex align-items-center">
												<span class="me-2">{result.avatar}</span>
												<strong>{result.name}</strong>
											</div>
										</td>
										{#if activityType === 'Probe'}
											<td>{result.accuracy}%</td>
											<td>{result.itemsCorrect}/{result.totalItems}</td>
											<td>{result.wcpm}</td>
										{:else if activityType === 'Diagnostic'}
											<td>{result.accuracy}%</td>
											<td>{result.errorPatterns}</td>
										{:else if activityType === 'Practice'}
											<td>{result.trials}</td>
											<td>{result.accuracy}%</td>
											<td>{result.improvement}</td>
										{:else if activityType === 'Extended Assignment'}
											<td>{result.completionRate}%</td>
											<td>{result.overallAccuracy}%</td>
											<td>{result.progress}</td>
										{/if}
										<td>
											<span class="badge bg-{getStatusColor(result.status)}">
												{result.status}
											</span>
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

	<!-- Class Recommendations -->
	<div class="row mt-4">
		<div class="col-12">
			<div class="card">
				<div class="card-body">
					<h5 class="card-title">Class-Level Recommendations</h5>
					<div class="row">
						<div class="col-md-4">
							<h6 class="text-success">Students Ready to Advance</h6>
							<ul class="list-unstyled">
								{#each studentResults.filter(r => r.status === 'Core') as student}
									<li>• {student.name}</li>
								{/each}
							</ul>
						</div>
						<div class="col-md-4">
							<h6 class="text-warning">Students Needing Additional Practice</h6>
							<ul class="list-unstyled">
								{#each studentResults.filter(r => r.status === 'Strategic') as student}
									<li>• {student.name}</li>
								{/each}
							</ul>
						</div>
						<div class="col-md-4">
							<h6 class="text-danger">Students Needing Intervention</h6>
							<ul class="list-unstyled">
								{#each studentResults.filter(r => r.status === 'Intensive') as student}
									<li>• {student.name}</li>
								{/each}
							</ul>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
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

	/* Dark header hover effect - use a dark color so white text is visible */
	.table-dark .sortable:hover {
		background-color: rgba(0, 0, 0, 0.3) !important;
		transform: scale(1.02) !important;
		transition: all 0.2s ease !important;
	}

	/* Even more specific for table headers */
	.table thead th.sortable:hover {
		background-color: rgba(0, 0, 0, 0.3) !important;
		transform: scale(1.02) !important;
		transition: all 0.2s ease !important;
	}

	/* Custom hover styles that preserve status colors */
	.table tbody tr:hover {
		opacity: 0.9;
		transform: scale(1.01);
		transition: all 0.2s ease;
	}

	.table tbody tr.table-success:hover {
		background-color: rgba(25, 135, 84, 0.1) !important;
	}

	.table tbody tr.table-warning:hover {
		background-color: rgba(255, 193, 7, 0.1) !important;
	}

	.table tbody tr.table-danger:hover {
		background-color: rgba(220, 53, 69, 0.1) !important;
	}

	@media print {
		.btn, .btn-group {
			display: none !important;
		}
	}
</style>