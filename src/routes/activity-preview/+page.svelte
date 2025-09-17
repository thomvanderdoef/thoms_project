<script lang="ts">
	import { onMount } from 'svelte';

	let activityType = '';
	let lessonFocus = '';
	let lessonTitle = '';

	// Sample class roster for assignment modal
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

	let showAssignmentModal = false;
	let selectedStudents: string[] = [];

	onMount(() => {
		// Get activity info from URL params
		const urlParams = new URLSearchParams(window.location.search);
		activityType = urlParams.get('activity') || 'Probe';
		lessonFocus = urlParams.get('lesson') || 'v/v/';
		lessonTitle = urlParams.get('lessonTitle') || 'Lesson';
	});

	function getActivityDescription(type: string) {
		switch (type) {
			case 'Probe':
				return 'Introduces the vowel teams \'ai\' and \'ay\' making the long /ā/ sound as in "rain," "play," "stay."';
			case 'Diagnostic':
				return 'Error pattern analysis for targeted intervention';
			case 'Practice':
				return 'Controlled repetition with immediate feedback';
			case 'Extended Assignment':
				return 'Multi-activity practice sequence';
			default:
				return 'Activity description';
		}
	}

	function openAssignmentModal() {
		selectedStudents = [];
		showAssignmentModal = true;
	}

	function closeAssignmentModal() {
		showAssignmentModal = false;
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

		// Close modal and navigate to assignment report
		closeAssignmentModal();

		// Create detailed assignment page
		const params = new URLSearchParams({
			activity: activityType,
			lesson: lessonFocus,
			students: selectedStudents.join(',')
		});

		window.location.href = `/assignment-report?${params.toString()}`;
	}

	function getPerformanceColor(performance: string) {
		switch (performance) {
			case 'Core': return 'success';
			case 'Strategic': return 'warning';
			case 'Intensive': return 'danger';
			default: return 'secondary';
		}
	}
</script>

<svelte:head>
	<title>{activityType}: {lessonFocus}</title>
</svelte:head>

<!-- Full height container matching Figma layout -->
<div class="container-fluid vh-100 p-0">
	<div class="row h-100 g-0">
		<!-- Left Sidebar - Activity Details (matches Figma left panel) -->
		<div class="col-md-4 col-lg-3 bg-white border-end">
			<div class="h-100 d-flex flex-column">
				<!-- Header Section -->
				<div class="p-4 border-bottom">
					<div class="d-flex justify-content-between align-items-center mb-3">
						<h4 class="mb-0 text-primary">{activityType}</h4>
						<button class="btn btn-outline-secondary btn-sm" on:click={() => history.back()}>
							CLOSE
						</button>
					</div>
					<p class="text-muted mb-0">{getActivityDescription(activityType)}</p>
				</div>

				<!-- Data Collection Section -->
				<div class="p-4 flex-grow-1">
					<h6 class="mb-3">Data Being Collected</h6>
					<div class="list-group list-group-flush">
						{#if activityType === 'Probe'}
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Accuracy percentage</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Words correct per minute (WCPM)</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Response time per word</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Error patterns and miscues</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Completion time</small>
							</div>
						{:else if activityType === 'Diagnostic'}
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Error pattern analysis</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Strength identification</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Gap analysis</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Intervention recommendations</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Skill mastery levels</small>
							</div>
						{:else if activityType === 'Practice'}
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Trial completion rate</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Improvement over time</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Error correction response</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Engagement metrics</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Time spent per trial</small>
							</div>
						{:else}
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Activity completion status</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Skill progression tracking</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Mastery level assessment</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Independence indicators</small>
							</div>
							<div class="list-group-item px-0 py-2 border-0">
								<small class="text-muted">• Overall performance score</small>
							</div>
						{/if}
					</div>
				</div>

				<!-- Assign Button at Bottom -->
				<div class="p-4 border-top">
					<button class="btn btn-primary w-100" on:click={openAssignmentModal}>
						Assign Activity
					</button>
				</div>
			</div>
		</div>

		<!-- Right Side - Activity Preview (simplified) -->
		<div class="col-md-8 col-lg-9 bg-light">
			<div class="h-100 d-flex align-items-center justify-content-center p-5">
				<div class="text-center">
					<div class="bg-white rounded-3 shadow-sm p-5" style="min-height: 400px; min-width: 500px;">
						<h5 class="text-muted mb-4">Activity Preview</h5>
						<div class="border rounded p-4 bg-light d-flex align-items-center justify-content-center" style="height: 300px;">
							<p class="text-muted mb-0">
								Interactive {activityType.toLowerCase()} interface
								<br><small>Student will interact with activity content here</small>
							</p>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>

<!-- Assignment Modal - Simplified to match Figma -->
{#if showAssignmentModal}
	<div class="modal fade show d-block" tabindex="-1" style="background-color: rgba(0,0,0,0.5);">
		<div class="modal-dialog modal-lg">
			<div class="modal-content">
				<div class="modal-header">
					<h5 class="modal-title">Assign {activityType} - {lessonFocus}</h5>
					<button type="button" class="btn-close" on:click={closeAssignmentModal}></button>
				</div>
				<div class="modal-body">
					<p class="text-muted mb-3">Select students to assign this activity to:</p>

					<!-- Quick Selection Buttons -->
					<div class="btn-group w-100 mb-3" role="group">
						<button
							class="btn btn-outline-secondary btn-sm"
							on:click={selectAllStudents}
						>
							All ({classRoster.length})
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

					<!-- Student Selection - Horizontal Layout matching Figma -->
					<div class="row g-2" style="max-height: 300px; overflow-y: auto;">
						{#each classRoster as student}
							<div class="col-lg-3 col-md-4 col-sm-6">
								<div
									class="bg-white border rounded p-2 {selectedStudents.includes(student.id) ? 'border-primary bg-light' : ''}"
									style="cursor: pointer; min-height: 60px;"
									on:click={() => toggleStudentSelection(student.id)}
								>
									<div class="d-flex align-items-center gap-2">
										<!-- Checkbox -->
										<input
											class="form-check-input"
											type="checkbox"
											checked={selectedStudents.includes(student.id)}
											on:change={() => toggleStudentSelection(student.id)}
											style="width: 24px; height: 24px;"
										>
										<!-- Student Name -->
										<div class="flex-grow-1">
											<div class="fw-medium small">{student.name}</div>
										</div>
										<!-- Performance Badge -->
										<span class="badge bg-{getPerformanceColor(student.performance)} rounded-pill">
											{student.performance}
										</span>
									</div>
								</div>
							</div>
						{/each}
					</div>

					<div class="mt-3">
						<small class="text-muted">
							{selectedStudents.length} student{selectedStudents.length !== 1 ? 's' : ''} selected
						</small>
					</div>
				</div>
				<div class="modal-footer">
					<button type="button" class="btn btn-outline-secondary" on:click={closeAssignmentModal}>
						Cancel
					</button>
					<button type="button" class="btn btn-outline-primary me-2" on:click={assignToSelectedStudents}>
						Schedule Activity
					</button>
					<button
						type="button"
						class="btn btn-primary"
						disabled={selectedStudents.length === 0}
						on:click={assignToSelectedStudents}
					>
						Assign Activity
					</button>
				</div>
			</div>
		</div>
	</div>
{/if}