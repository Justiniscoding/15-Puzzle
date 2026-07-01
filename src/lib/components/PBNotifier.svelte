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

<!-- svelte-ignore a11y_click_events_have_key_events -->
<!-- svelte-ignore a11y_no_static_element_interactions -->
<div class="container" onclick={() => onFinished(index)}>
	<h1>New PB!</h1>
	<p>
		<span class="red">{formatNumber(oldTime)}s</span>
		→
		<span class="green">{formatNumber(newTime)}s</span>
	</p>
	<p>{formatNumber(oldTime - newTime)}s faster</p>
	<progress value={progressAmount} max={animationTime}></progress>
</div>

<style>
	.container {
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;

		background-color: black;
		border: 7px solid #39ff14;
		border-radius: 3vw;

		width: 40vw;

		padding: 1vw 2vw;

		position: absolute;

		top: 2vh;
		left: 50vw;

		transform: translate(-50%, 0);
	}

	h1 {
		font-family: Arial, Helvetica, sans-serif;
	}

	p,
	h1 {
		color: white;
		margin: 0.2vh;
	}

	.red {
		color: #ff0000;
		font-weight: bold;
	}

	.green {
		color: #39ff14;
		font-weight: bold;
	}
</style>
