<script lang="ts">
	import ufliLessons from '$lib/data/ufli-lessons.json';

	let selectedLesson: any = null;
	let selectedResults: any = null;
	let showAssignmentModal = false;
	let selectedActivityType = '';
	let selectedStudents: string[] = [];

	// Sample class roster
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

	function selectLesson(lesson: any) {
		selectedLesson = lesson;
		selectedResults = null; // Clear previous results
	}

	function generateActivity(activityType: string) {
		if (!selectedLesson) {
			alert('Please select a lesson first');
			return;
		}

		// Generate fake student results
		const fakeResults = generateFakeResults(activityType, selectedLesson);

		// Open activity in new tab
		const activityWindow = window.open('', '_blank');
		if (activityWindow) {
			activityWindow.document.write(generateActivityHTML(activityType, selectedLesson, fakeResults));

			// Simulate completion after 3 seconds and show results
			setTimeout(() => {
				selectedResults = fakeResults;
				activityWindow.close();
			}, 3000);
		}
	}

	function viewClassResults(activityType: string) {
		if (!selectedLesson) {
			alert('Please select a lesson first');
			return;
		}

		// Open class results page in new tab
		const params = new URLSearchParams({
			activity: activityType,
			lesson: selectedLesson.lessonFocus
		});
		window.open(`/class-results?${params.toString()}`, '_blank');
	}

	function openActivityPreview(activityType: string) {
		if (!selectedLesson) {
			alert('Please select a lesson first');
			return;
		}

		// Navigate to activity preview page
		const params = new URLSearchParams({
			activity: activityType,
			lesson: selectedLesson.lessonFocus,
			lessonTitle: selectedLesson.lesson
		});
		window.location.href = `/activity-preview?${params.toString()}`;
	}

	function openAssignmentModal(activityType: string) {
		selectedActivityType = activityType;
		selectedStudents = [];
		showAssignmentModal = true;
	}

	function closeAssignmentModal() {
		showAssignmentModal = false;
		selectedActivityType = '';
		selectedStudents = [];
	}

	function toggleStudentSelection(studentId: string) {
		if (selectedStudents.includes(studentId)) {
			selectedStudents = selectedStudents.filter(id => id !== studentId);
		} else {
			selectedStudents = [...selectedStudents, studentId];
		}
	}

	function selectAllStudents() {
		selectedStudents = classRoster.map(student => student.id);
	}

	function selectByPerformance(performance: string) {
		selectedStudents = classRoster
			.filter(student => student.performance === performance)
			.map(student => student.id);
	}

	function assignToSelectedStudents() {
		if (selectedStudents.length === 0) {
			alert('Please select at least one student');
			return;
		}

		// Close modal and show assignment confirmation
		closeAssignmentModal();

		// Create detailed assignment page
		const params = new URLSearchParams({
			activity: selectedActivityType,
			lesson: selectedLesson.lessonFocus,
			students: selectedStudents.join(',')
		});
		window.open(`/assignment-report?${params.toString()}`, '_blank');
	}

	function getPerformanceColor(performance: string) {
		switch (performance) {
			case 'Core': return 'success';
			case 'Strategic': return 'warning';
			case 'Intensive': return 'danger';
			default: return 'secondary';
		}
	}

	function generateFakeResults(activityType: string, lesson: any) {
		// Generate realistic fake data based on activity type
		const baseAccuracy = Math.random() * 0.4 + 0.6; // 60-100% accuracy

		switch (activityType) {
			case 'Probe':
				return {
					activityType: 'Probe',
					lessonFocus: lesson.lessonFocus,
					accuracy: Math.round(baseAccuracy * 100),
					itemsCorrect: Math.round(baseAccuracy * 10),
					totalItems: 10,
					avgLatency: Math.round(Math.random() * 1000 + 1000), // 1-2 seconds
					wcpm: Math.round(Math.random() * 20 + 40), // 40-60 WCPM
					miscues: ['substitution: b/d confusion', 'omission: final consonant'],
					recommendation: baseAccuracy > 0.8 ? 'Continue current instruction' : 'Consider additional practice'
				};
			case 'Diagnostic':
				return {
					activityType: 'Diagnostic',
					lessonFocus: lesson.lessonFocus,
					accuracy: Math.round(baseAccuracy * 100),
					errorPatterns: [
						{ pattern: 'Letter reversal (b/d)', frequency: 3 },
						{ pattern: 'Short vowel confusion', frequency: 2 }
					],
					strengthAreas: ['Initial consonants', 'Blending CVC words'],
					needsWork: ['Final consonants', 'Vowel discrimination'],
					intervention: 'Focus on letter formation and vowel sounds'
				};
			case 'Practice':
				return {
					activityType: 'Practice',
					lessonFocus: lesson.lessonFocus,
					trialsCompleted: 15,
					accuracy: Math.round(baseAccuracy * 100),
					improvement: '+12% from first attempt',
					timeSpent: '8 minutes',
					feedback: 'Student showed improvement with repeated practice'
				};
			case 'Assignment':
				return {
					activityType: 'Assignment',
					lessonFocus: lesson.lessonFocus,
					activitiesCompleted: 4,
					totalActivities: 5,
					overallAccuracy: Math.round(baseAccuracy * 100),
					timeSpent: '18 minutes',
					status: 'In Progress'
				};
			default:
				return {};
		}
	}

	function generateActivityHTML(activityType: string, lesson: any, results: any) {
		return `
			<!DOCTYPE html>
			<html>
			<head>
				<title>${activityType} - ${lesson.lessonFocus}</title>
				<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
				<style>
					body { padding: 2rem; background: #f8f9fa; }
					.activity-card { background: white; border-radius: 12px; padding: 2rem; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
				</style>
			</head>
			<body>
				<div class="container">
					<div class="activity-card">
						<h2 class="text-primary mb-4">${activityType}: ${lesson.lessonFocus}</h2>
						<p class="lead">${lesson.lessonDescription}</p>

						<div class="mt-4">
							<h4>Student Activity Simulation</h4>
							<div class="border rounded p-3 bg-light">
								<div class="text-center">
									<div class="spinner-border text-primary" role="status">
										<span class="visually-hidden">Loading...</span>
									</div>
									<p class="mt-3">Simulating student interaction...</p>
									<p class="text-muted">This will complete automatically in 3 seconds</p>
								</div>
							</div>
						</div>

						<div class="mt-4">
							<h5>Expected Results Preview:</h5>
							<ul class="list-group">
								${Object.entries(results).map(([key, value]) =>
									`<li class="list-group-item d-flex justify-content-between">
										<strong>${key}:</strong> <span>${value}</span>
									</li>`
								).join('')}
							</ul>
						</div>
					</div>
				</div>
			</body>
			</html>
		`;
	}
</script>

<div class="container py-5">
	<div class="row">
		<div class="col-12 text-center mb-5">
			<h1 class="display-4 text-primary">Science of Reading Activity Generator</h1>
			<p class="lead text-muted">Select a UFLI lesson to get started</p>
		</div>
	</div>

	<div class="row">
		<div class="col-lg-8 mx-auto">
			<!-- Step 1: Lesson Selection (Always Visible) -->
			<div class="card mb-4">
				<div class="card-body">
					<h5 class="card-title mb-3">Choose a lesson</h5>
					<select class="form-select lesson-selector" on:change={(e) => selectLesson(ufliLessons[parseInt(e.target.value)])}>
						<option value="">Choose a lesson...</option>
						{#each ufliLessons as lesson, index}
							<option value={index}>
								{lesson.lesson}: {lesson.lessonFocus} - {lesson.unit}
							</option>
						{/each}
					</select>
				</div>
			</div>

			<!-- Step 2: Lesson Description (Only show after lesson selection) -->
			{#if selectedLesson}
				<div class="card mb-4">
					<div class="card-body">
						<h5 class="card-title mb-3">Lesson Description</h5>
						<div class="p-3 bg-light rounded">
							<h6 class="mb-2">{selectedLesson.lesson}: {selectedLesson.lessonFocus}</h6>
							<p class="mb-0 text-muted">{selectedLesson.lessonDescription}</p>
						</div>
					</div>
				</div>

				<!-- Step 3: Activity Options (Only show after lesson selection) -->
				<div class="card mb-4">
					<div class="card-body">
						<h5 class="card-title mb-3">Choose activity</h5>
						<div class="row g-3">
							<div class="col-md-6">
								<button
									class="btn btn-primary btn-activity w-100 py-3"
									on:click={() => openActivityPreview('Probe')}
								>
									<h6 class="mb-1">Probe</h6>
									<small>Simple explanation of when and why to assign this, based on selected lesson.</small>
								</button>
							</div>
							<div class="col-md-6">
								<button
									class="btn btn-success btn-activity w-100 py-3"
									on:click={() => openActivityPreview('Diagnostic')}
								>
									<h6 class="mb-1">Diagnostic</h6>
									<small>Simple explanation of when and why to assign this, based on selected lesson.</small>
								</button>
							</div>
							<div class="col-md-6">
								<button
									class="btn btn-warning btn-activity w-100 py-3"
									on:click={() => openActivityPreview('Practice')}
								>
									<h6 class="mb-1">Practice</h6>
									<small>Simple explanation of when and why to assign this, based on selected lesson.</small>
								</button>
							</div>
							<div class="col-md-6">
								<button
									class="btn btn-info btn-activity w-100 py-3"
									on:click={() => openActivityPreview('Extended Assignment')}
								>
									<h6 class="mb-1">Extended Assignment</h6>
									<small>Simple explanation of when and why to assign this, based on selected lesson.</small>
								</button>
							</div>
						</div>
					</div>
				</div>

			{/if}

			{#if selectedResults}
				<div class="card">
					<div class="card-body">
						<h5 class="card-title text-success mb-3">
							Activity Results: {selectedResults.activityType}
						</h5>

						<div class="row">
							<div class="col-md-6">
								<h6>Performance Metrics</h6>
								{#if selectedResults.accuracy}
									<p><strong>Accuracy:</strong> {selectedResults.accuracy}%</p>
								{/if}
								{#if selectedResults.itemsCorrect}
									<p><strong>Items Correct:</strong> {selectedResults.itemsCorrect}/{selectedResults.totalItems}</p>
								{/if}
								{#if selectedResults.wcpm}
									<p><strong>WCPM:</strong> {selectedResults.wcpm}</p>
								{/if}
								{#if selectedResults.timeSpent}
									<p><strong>Time Spent:</strong> {selectedResults.timeSpent}</p>
								{/if}
							</div>

							<div class="col-md-6">
								<h6>Analysis</h6>
								{#if selectedResults.miscues}
									<p><strong>Miscues:</strong></p>
									<ul class="list-unstyled ms-3">
										{#each selectedResults.miscues as miscue}
											<li>• {miscue}</li>
										{/each}
									</ul>
								{/if}
								{#if selectedResults.errorPatterns}
									<p><strong>Error Patterns:</strong></p>
									<ul class="list-unstyled ms-3">
										{#each selectedResults.errorPatterns as error}
											<li>• {error.pattern} ({error.frequency}x)</li>
										{/each}
									</ul>
								{/if}
								{#if selectedResults.recommendation}
									<div class="alert alert-info mt-3">
										<strong>Recommendation:</strong> {selectedResults.recommendation}
									</div>
								{/if}
								{#if selectedResults.intervention}
									<div class="alert alert-warning mt-3">
										<strong>Intervention:</strong> {selectedResults.intervention}
									</div>
								{/if}
							</div>
						</div>
					</div>
				</div>
			{/if}
		</div>
	</div>
</div>

<!-- Assignment Modal -->
{#if showAssignmentModal}
	<div class="modal fade show d-block" tabindex="-1" style="background-color: rgba(0,0,0,0.5);">
		<div class="modal-dialog modal-lg">
			<div class="modal-content">
				<div class="modal-header">
					<h5 class="modal-title">Assign {selectedActivityType} - {selectedLesson?.lessonFocus}</h5>
					<button type="button" class="btn-close" on:click={closeAssignmentModal}></button>
				</div>
				<div class="modal-body">
					<div class="row mb-3">
						<div class="col-12">
							<p class="text-muted">Select students to assign this activity to:</p>
						</div>
					</div>

					<!-- Quick Selection Buttons -->
					<div class="row mb-3">
						<div class="col-12">
							<div class="btn-group me-2" role="group">
								<button
									class="btn btn-outline-secondary btn-sm"
									on:click={selectAllStudents}
								>
									Select All ({classRoster.length})
								</button>
								<button
									class="btn btn-outline-success btn-sm"
									on:click={() => selectByPerformance('Core')}
								>
									Core ({classRoster.filter(s => s.performance === 'Core').length})
								</button>
								<button
									class="btn btn-outline-warning btn-sm"
									on:click={() => selectByPerformance('Strategic')}
								>
									Strategic ({classRoster.filter(s => s.performance === 'Strategic').length})
								</button>
								<button
									class="btn btn-outline-danger btn-sm"
									on:click={() => selectByPerformance('Intensive')}
								>
									Intensive ({classRoster.filter(s => s.performance === 'Intensive').length})
								</button>
							</div>
						</div>
					</div>

					<!-- Student Selection Grid -->
					<div class="row">
						{#each classRoster as student}
							<div class="col-md-6 col-lg-4 mb-2">
								<div
									class="card h-100 {selectedStudents.includes(student.id) ? 'border-primary bg-light' : ''}"
									style="cursor: pointer;"
									on:click={() => toggleStudentSelection(student.id)}
								>
									<div class="card-body p-2">
										<div class="form-check">
											<input
												class="form-check-input"
												type="checkbox"
												checked={selectedStudents.includes(student.id)}
												on:change={() => toggleStudentSelection(student.id)}
											>
											<label class="form-check-label d-flex align-items-center">
												<span class="me-2" style="font-size: 1.2em;">{student.avatar}</span>
												<div>
													<div class="fw-bold small">{student.name}</div>
													<span class="badge bg-{getPerformanceColor(student.performance)} small">
														{student.performance}
													</span>
												</div>
											</label>
										</div>
									</div>
								</div>
							</div>
						{/each}
					</div>

					<!-- Assignment Settings -->
					<div class="row mt-4">
						<div class="col-md-6">
							<label class="form-label">Due Date</label>
							<input type="date" class="form-control" value={new Date().toISOString().split('T')[0]}>
						</div>
						<div class="col-md-6">
							<label class="form-label">Assignment Mode</label>
							<select class="form-select">
								<option>Immediate Assignment</option>
								<option>Scheduled for Later</option>
							</select>
						</div>
					</div>
				</div>
				<div class="modal-footer">
					<div class="me-auto">
						<small class="text-muted">
							{selectedStudents.length} student{selectedStudents.length !== 1 ? 's' : ''} selected
						</small>
					</div>
					<button type="button" class="btn btn-secondary" on:click={closeAssignmentModal}>
						Cancel
					</button>
					<button
						type="button"
						class="btn btn-primary"
						disabled={selectedStudents.length === 0}
						on:click={assignToSelectedStudents}
					>
						Assign to {selectedStudents.length} Student{selectedStudents.length !== 1 ? 's' : ''}
					</button>
				</div>
			</div>
		</div>
	</div>
{/if}
