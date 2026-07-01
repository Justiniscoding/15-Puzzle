<script lang="ts">
	import { onMount } from "svelte";

	let { oldTime, newTime, onFinished } = $props();

	let progressAmount = $state(0);

	let lastFrame: number;

	onMount(() => {
		loop();
	});

	let frame: number;

	function loop() {
		if (lastFrame) {
			const deltaTime = Date.now() - lastFrame;

			progressAmount += deltaTime;
		}

		if (progressAmount > 3000) {
			cancelAnimationFrame(frame);
		}

		lastFrame = Date.now();
		frame = requestAnimationFrame(loop);
	}
</script>

<div class="container">
	<h1>New PB!</h1>
	<p>{oldTime} -> {newTime} = -{newTime - oldTime}</p>
	<progress value={progressAmount} max="3000"></progress>
</div>
