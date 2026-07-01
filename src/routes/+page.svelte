<script lang="ts">
	import SlidingPuzzle from "$lib/components/SlidingPuzzle.svelte";
	import { onMount } from "svelte";

	let dialogElement: HTMLDialogElement;

	function closeModal() {
		dialogElement.close();
	}

	onMount(() => {
		if (!localStorage.hasSeenTutorial) {
			localStorage.hasSeenTutorial = true;

			dialogElement.showModal();
		}
	});
</script>

<div class="container">
	<h1>15 Puzzle</h1>
	<SlidingPuzzle />
</div>

<button id="questionMark" onclick={() => dialogElement.showModal()}>?</button>

<dialog bind:this={dialogElement} id="tutorial">
	<div class="dialogContainer">
		<h1>Tutorial</h1>
		<p>
			The 15 puzzle is a sliding puzzle where you have to get numbers from
			1-15 in order in a 4x4 grid. This is done by sliding a tile that is
			directly adjacent to the blank position into its place. A timer
			starts after you make your first move, and the amount of moves you
			make per second is kept track of as well.
		</p>
		<button onclick={closeModal}>Close</button>
	</div>
</dialog>

<style>
	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		margin: 0;
		padding: 0;
	}

	h1 {
		font-family: Arial, Helvetica, sans-serif;
		font-size: 3vw;
	}

	:global(html, body) {
		background-color: #aaa;
		margin: 0;
		padding: 0;
	}

	.dialogContainer {
		width: 70vw;
		display: flex;
		align-items: center;
		flex-direction: column;
	}

	.dialogContainer > h1 {
		font-size: 5vw;
	}

	.dialogContainer > p {
		font-size: 3vw;
		text-align: left;
	}

	#questionMark {
		border-radius: 50%;
		background: none;
		position: absolute;
		margin: 0;
		top: 1vw;
		right: 1vw;
		padding: 0;
		text-align: center;
		width: 3vw;
		height: 3vw;
		font-size: 2vw;
	}
</style>
