<script lang="ts">
	import SlidingPuzzle from "$lib/components/SlidingPuzzle.svelte";
	import { onMount } from "svelte";

	// import { formatNumber } from "$lib/components/PBNotifier.svelte";

	let dialogElement: HTMLDialogElement;

	let showingLeaderboard: boolean = $state(false);
	let numTiles: number = $state(16);

	let personalBests: Map<number, number> = new Map();

	function closeModal() {
		dialogElement.close();
	}

	onMount(() => {
		if (!localStorage.hasSeenTutorial) {
			localStorage.hasSeenTutorial = true;

			dialogElement.showModal();
		}
	});

	function showModal(isLeaderboard: boolean) {
		if (isLeaderboard) {
			for (let i = 2; i <= 20; i++) {
				const key = `${i * i - 1}_puzzle_pb`;
				if (localStorage[key]) {
					personalBests.set(i * i - 1, localStorage[key]);
				}
			}
		}

		showingLeaderboard = isLeaderboard;
		dialogElement.showModal();
	}
</script>

<div class="container">
	<h1>{numTiles - 1} Puzzle</h1>
	<SlidingPuzzle bind:numTiles />
</div>

<button id="questionMark" onclick={() => showModal(false)}>?</button>
<button id="leaderboard" onclick={() => showModal(true)}>🏆</button>

<dialog bind:this={dialogElement} id="tutorial">
	<div class="dialogContainer">
		{#if showingLeaderboard}
			<h1>Personal Bests</h1>
			{#each personalBests.entries() as item}
				<p>
					<span style="font-weight: 900;">{item[0]} Puzzle</span> -
					<span style="color: #016002; font-weight:700;"
						>{(item[1] / 1000).toFixed(3)}s</span
					>
				</p>
			{:else}
				<p>You currently have no personal bests. Go do some solves!</p>
			{/each}
		{:else}
			<h1>Tutorial</h1>
			<p>
				The 15 puzzle is a sliding puzzle where you have to get numbers
				from 1-15 in order in a 4x4 grid. This is done by sliding a tile
				that is directly adjacent to the blank position into its place.
				A timer starts after you make your first move, and the amount of
				moves you make per second is kept track of as well.
			</p>
		{/if}
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

	#questionMark,
	#leaderboard {
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

	.dialogContainer > p {
		margin: 1vw;
	}

	#leaderboard {
		left: 1vw;
	}
</style>
