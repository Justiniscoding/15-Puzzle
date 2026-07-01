<script lang="ts">
	import SlidingPuzzle from "$lib/components/SlidingPuzzle.svelte";
	import { onMount } from "svelte";

	let dialogElement: HTMLDialogElement;

	let numTiles: number = $state(16);

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
	<h1>{numTiles - 1} Puzzle</h1>
	<SlidingPuzzle bind:numTiles />
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

	dialog {
		border-radius: 2.5vw;
		background-color: lightgoldenrodyellow;
		transition: 0.35s;
	}

	dialog[open] {
		transform: scale(1);
	}

	@starting-style {
		dialog[open] {
			transform: scale(0);
		}
	}

	dialog::backdrop {
		background-color: rgba(0, 0, 0, 0.3);
	}

	.dialogContainer {
		width: min(70vw, 70vh);
		display: flex;
		align-items: center;
		flex-direction: column;
	}

	.dialogContainer > h1 {
		font-size: min(5vw, 5vh);
	}

	.dialogContainer > p {
		font-size: min(3vw, 3vh);
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
