<script lang="ts">
	import { onMount } from 'svelte';

	let currentLetter = 'v';
	let currentStep = 1;
	let totalSteps = 3;
	let isRecording = false;
	let recordingComplete = false;
	let audioResult = null;
	let showFeedback = false;
	let allResults = [];

	const letters = ['v', 'b', 'f'];
	const expectedSounds = {
		'v': '/v/',
		'b': '/b/',
		'f': '/f/'
	};

	onMount(() => {
		currentLetter = letters[currentStep - 1];
	});

	function startRecording() {
		isRecording = true;
		recordingComplete = false;
		showFeedback = false;

		// Simulate 2-second recording
		setTimeout(() => {
			isRecording = false;
			recordingComplete = true;
			processAudio();
		}, 2000);
	}

	function processAudio() {
		// Simulate audio processing delay
		setTimeout(() => {
			// Generate realistic results based on letter
			const isCorrect = Math.random() > 0.2; // 80% correct rate
			const accuracy = isCorrect ? Math.floor(Math.random() * 20) + 80 : Math.floor(Math.random() * 60) + 20;

			audioResult = {
				letter: currentLetter,
				expectedSound: expectedSounds[currentLetter],
				accuracy: accuracy,
				isCorrect: isCorrect,
				detectedSound: isCorrect ? expectedSounds[currentLetter] : getRandomMiscue(currentLetter),
				feedback: isCorrect ? 'Great job!' : 'Let\'s try that again.'
			};

			allResults.push(audioResult);
			showFeedback = true;
		}, 1500);
	}

	function getRandomMiscue(letter) {
		const miscues = {
			'v': ['/b/', '/f/', '/w/'],
			'b': ['/v/', '/p/', '/d/'],
			'f': ['/v/', '/th/', '/p/']
		};
		const options = miscues[letter] || ['/uh/'];
		return options[Math.floor(Math.random() * options.length)];
	}

	function nextLetter() {
		if (currentStep < totalSteps) {
			currentStep++;
			currentLetter = letters[currentStep - 1];
			recordingComplete = false;
			showFeedback = false;
			audioResult = null;
		} else {
			// Activity complete - show results
			currentStep = totalSteps + 1;
		}
	}

	function restartActivity() {
		currentStep = 1;
		currentLetter = letters[0];
		recordingComplete = false;
		showFeedback = false;
		audioResult = null;
		allResults = [];
	}

	function getOverallAccuracy() {
		if (allResults.length === 0) return 0;
		return Math.round(allResults.reduce((sum, result) => sum + result.accuracy, 0) / allResults.length);
	}

	function getCorrectCount() {
		return allResults.filter(result => result.isCorrect).length;
	}
</script>

<svelte:head>
	<title>Letter Sound Practice</title>
</svelte:head>

<div class="container-fluid vh-100 d-flex align-items-center justify-content-center" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);">
	<div class="row w-100 justify-content-center">
		<div class="col-12 col-md-8 col-lg-6">
			{#if currentStep <= totalSteps}
				<!-- Activity Card -->
				<div class="card shadow-lg border-0">
					<div class="card-body text-center p-5">
						<!-- Progress Bar -->
						<div class="mb-4">
							<div class="progress" style="height: 8px;">
								<div class="progress-bar bg-success" role="progressbar"
									style="width: {(currentStep / totalSteps) * 100}%"></div>
							</div>
							<small class="text-muted mt-2 d-block">Step {currentStep} of {totalSteps}</small>
						</div>

						<!-- Letter Display -->
						<div class="mb-4">
							<h2 class="text-muted mb-2">What sound does this letter make?</h2>
							<div class="display-1 text-primary fw-bold mb-3" style="font-size: 8rem;">
								{currentLetter}
							</div>
						</div>

						<!-- Recording Interface -->
						{#if !recordingComplete}
							<div class="mb-4">
								{#if !isRecording}
									<button class="btn btn-primary btn-lg px-5 py-3" on:click={startRecording}>
										<i class="bi bi-mic-fill me-2"></i>
										Press to Say the Sound
									</button>
									<p class="text-muted mt-3 mb-0">Click the button and say the sound this letter makes</p>
								{:else}
									<div class="recording-animation mb-3">
										<div class="btn btn-danger btn-lg px-5 py-3 disabled">
											<i class="bi bi-mic-fill me-2"></i>
											Recording...
										</div>
										<div class="recording-pulse mt-3"></div>
									</div>
									<p class="text-info mt-3 mb-0">Listening... Speak clearly!</p>
								{/if}
							</div>
						{:else if showFeedback && audioResult}
							<!-- Feedback -->
							<div class="mb-4">
								<div class="alert alert-{audioResult.isCorrect ? 'success' : 'warning'} p-4">
									<h4 class="alert-heading">
										{#if audioResult.isCorrect}
											🎉 {audioResult.feedback}
										{:else}
											🤔 {audioResult.feedback}
										{/if}
									</h4>
									<div class="mt-3">
										<strong>You said:</strong> {audioResult.detectedSound}<br>
										<strong>Expected:</strong> {audioResult.expectedSound}<br>
										<strong>Accuracy:</strong> {audioResult.accuracy}%
									</div>
								</div>

								<button class="btn btn-primary btn-lg px-4" on:click={nextLetter}>
									{#if currentStep < totalSteps}
										Next Letter <i class="bi bi-arrow-right ms-2"></i>
									{:else}
										See Results <i class="bi bi-bar-chart ms-2"></i>
									{/if}
								</button>
							</div>
						{:else}
							<!-- Processing -->
							<div class="mb-4">
								<div class="spinner-border text-primary mb-3" role="status">
									<span class="visually-hidden">Processing...</span>
								</div>
								<p class="text-muted">Analyzing your pronunciation...</p>
							</div>
						{/if}
					</div>
				</div>
			{:else}
				<!-- Results Summary -->
				<div class="card shadow-lg border-0">
					<div class="card-body text-center p-5">
						<h2 class="text-primary mb-4">🎯 Activity Complete!</h2>

						<!-- Overall Stats -->
						<div class="row mb-4">
							<div class="col-md-4">
								<div class="card bg-light border-0">
									<div class="card-body">
										<h3 class="text-success display-6 mb-0">{getCorrectCount()}/{totalSteps}</h3>
										<small class="text-muted">Correct</small>
									</div>
								</div>
							</div>
							<div class="col-md-4">
								<div class="card bg-light border-0">
									<div class="card-body">
										<h3 class="text-primary display-6 mb-0">{getOverallAccuracy()}%</h3>
										<small class="text-muted">Accuracy</small>
									</div>
								</div>
							</div>
							<div class="col-md-4">
								<div class="card bg-light border-0">
									<div class="card-body">
										<h3 class="text-info display-6 mb-0">{allResults.length}</h3>
										<small class="text-muted">Attempts</small>
									</div>
								</div>
							</div>
						</div>

						<!-- Detailed Results -->
						<div class="mb-4">
							<h5 class="mb-3">Your Results:</h5>
							<div class="table-responsive">
								<table class="table table-sm">
									<thead class="table-light">
										<tr>
											<th>Letter</th>
											<th>Your Sound</th>
											<th>Expected</th>
											<th>Result</th>
										</tr>
									</thead>
									<tbody>
										{#each allResults as result}
											<tr class="table-{result.isCorrect ? 'success' : 'warning'}">
												<td><strong>{result.letter}</strong></td>
												<td>{result.detectedSound}</td>
												<td>{result.expectedSound}</td>
												<td>
													{#if result.isCorrect}
														✅ Correct
													{:else}
														⚠️ Try Again
													{/if}
												</td>
											</tr>
										{/each}
									</tbody>
								</table>
							</div>
						</div>

						<!-- Action Buttons -->
						<div class="d-flex gap-3 justify-content-center">
							<button class="btn btn-outline-primary" on:click={restartActivity}>
								<i class="bi bi-arrow-repeat me-2"></i>
								Try Again
							</button>
							<button class="btn btn-primary" on:click={() => window.close()}>
								<i class="bi bi-check-circle me-2"></i>
								Finish
							</button>
						</div>
					</div>
				</div>
			{/if}
		</div>
	</div>
</div>

<style>
	.recording-pulse {
		width: 100px;
		height: 100px;
		border-radius: 50%;
		background: rgba(220, 53, 69, 0.3);
		margin: 0 auto;
		animation: pulse 1.5s ease-in-out infinite;
	}

	@keyframes pulse {
		0% {
			transform: scale(0.8);
			opacity: 1;
		}
		50% {
			transform: scale(1.2);
			opacity: 0.7;
		}
		100% {
			transform: scale(0.8);
			opacity: 1;
		}
	}

	.recording-animation {
		animation: bounce 0.5s ease-in-out infinite alternate;
	}

	@keyframes bounce {
		0% { transform: translateY(0); }
		100% { transform: translateY(-10px); }
	}
</style>