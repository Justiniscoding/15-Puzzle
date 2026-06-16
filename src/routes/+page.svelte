<script lang="ts">
	import SlidingPuzzle from "$lib/components/SlidingPuzzle.svelte";
</script>

<div class="container">
	<h1>15 Puzzle</h1>
	<SlidingPuzzle />
</div>

<style>
	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	:global(html, body) {
		background-color: #aaa;
	}
</style>
