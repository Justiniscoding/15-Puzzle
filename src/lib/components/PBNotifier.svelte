<script lang="ts">
	import { onMount } from "svelte";

	let { oldTime, newTime, onFinished, index } = $props();

	const animationTime = 8000;

	let progressAmount = $state(animationTime);

	let lastFrame: number;

	onMount(() => {
		loop();
	});

	let frame: number;

	function loop() {
		if (lastFrame) {
			const deltaTime = Date.now() - lastFrame;

			progressAmount -= deltaTime;
		}

		if (progressAmount < 0) {
			onFinished(index);
			cancelAnimationFrame(frame);
		}

		lastFrame = Date.now();
		frame = requestAnimationFrame(loop);
	}

	function formatNumber(msNumber: number) {
		return (msNumber / 1000).toFixed(3);
	}
</script>

<div class="container">
	<h1>New PB!</h1>
	<p>
		<span class="red">{formatNumber(oldTime)}s</span>
		→
		<span class="green">{formatNumber(newTime)}s</span>
	</p>
	<p>{oldTime - newTime} seconds faster</p>
	<progress value={progressAmount} max={animationTime}></progress>
</div>

<style>
	.container {
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;

		background-color: black;
		border: 10px solid #39ff14;
		border-radius: 3vw;

		width: 40vw;
		height: 20vh;

		position: absolute;

		bottom: 5vh;
		left: 50vw;

		transform: translate(-50%, 0);
	}

	h1 {
		font-family: Arial, Helvetica, sans-serif;
		color: white;
	}

	p {
		color: white;
	}

	.red {
		color: red;
		font-weight: bold;
	}

	.green {
		color: green;
		font-weight: bold;
	}
</style>
