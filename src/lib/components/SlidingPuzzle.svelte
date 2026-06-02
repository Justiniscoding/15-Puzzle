<script lang="ts">
	import { onMount } from "svelte";

	let canvas: HTMLCanvasElement;
	let context: CanvasRenderingContext2D | null;

	let tileSize: number;

	const tilesPerRow = 4;
	const numTiles = Math.pow(tilesPerRow, 2);

	let blankTileX = tilesPerRow - 1;
	let blankTileY = tilesPerRow - 1;

	let movesDone = $state(0);

	let gameGrid: number[] = new Array(numTiles).fill(null).map((el, index) => {
		if (index == numTiles - 1) {
			return -1;
		}
		return index + 1;
	});

	onMount(() => {
		context = canvas.getContext("2d");

		canvas.width = Math.min(innerWidth, innerHeight) - 200;
		canvas.height = Math.min(innerWidth, innerHeight) - 200;

		tileSize = canvas.width / tilesPerRow;

		shuffleGrid();

		loop();
	});

	function loop() {
		if (context == null) {
			return;
		}

		context.fillStyle = "black";
		context.fillRect(0, 0, canvas.width, canvas.height);

		for (let i = 0; i < gameGrid.length; i++) {
			if (gameGrid[i] == -1) {
				continue;
			}

			const x = (i % tilesPerRow) * tileSize;
			const y = Math.floor(i / tilesPerRow) * tileSize;

			const hue = (360 / (numTiles - 1)) * (gameGrid[i] - 1);

			context.fillStyle = `hsl(${hue}, 50%, 50%)`;
			context.fillRect(x, y, tileSize, tileSize);

			context.font = "50px Arial";
			context.textBaseline = "middle";
			context.textAlign = "center";

			context.fillStyle = "black";

			context.fillText(
				gameGrid[i].toString(),
				x + tileSize / 2,
				y + tileSize / 2,
			);
		}

		requestAnimationFrame(loop);
	}

	function swapTileWithBlank(tileX: number, tileY: number) {
		const tileIndex = tileY * tilesPerRow + tileX;
		const blankIndex = blankTileY * tilesPerRow + blankTileX;

		blankTileX = tileX;
		blankTileY = tileY;

		const temp = gameGrid[tileIndex];

		gameGrid[tileIndex] = gameGrid[blankIndex];
		gameGrid[blankIndex] = temp;
	}

	function swapTilesOnClick(event: MouseEvent) {
		// TODO: use getBoundingClientRect to get the offset of the canvas
		// relative to the DOM (or a more effecient method) in the future.

		const tileX = Math.floor(event.x / tileSize);
		const tileY = Math.floor(event.y / tileSize);

		let xOffsets = [1, -1, 0, 0];
		let yOffsets = [0, 0, 1, -1];

		for (let i = 0; i < 4; i++) {
			if (
				tileX + xOffsets[i] == blankTileX &&
				tileY + yOffsets[i] == blankTileY
			) {
				swapTileWithBlank(tileX, tileY);
				movesDone++;
				return;
			}
		}
	}

	async function shuffleGrid() {
		let xOffsets = [1, -1, 0, 0];
		let yOffsets = [0, 0, 1, -1];

		for (let i = 0; i < Math.pow(numTiles, 2); i++) {
			const offsetIndex = Math.floor(Math.random() * 4);

			const tileX = blankTileX + xOffsets[offsetIndex];
			const tileY = blankTileY + yOffsets[offsetIndex];

			if (
				tileX < 0 ||
				tileY < 0 ||
				tileX == tilesPerRow ||
				tileY == tilesPerRow
			) {
				continue;
			}

			// await new Promise((resolve) => setTimeout(resolve, 1));

			swapTileWithBlank(tileX, tileY);
		}
	}
</script>

<canvas bind:this={canvas} onclick={swapTilesOnClick}></canvas>

<h2>Moves: {movesDone}</h2>
