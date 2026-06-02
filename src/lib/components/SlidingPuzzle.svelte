<script lang="ts">
	import { onMount } from "svelte";

	let canvas: HTMLCanvasElement;
	let context: CanvasRenderingContext2D | null;

	let tileSize: number;

	const numTiles = 16;
	const tilesPerRow = Math.sqrt(numTiles);

	let gameGrid = new Array(numTiles).fill(null).map((el, index) => {
		if (index == numTiles - 1) {
			return null;
		}
		return index + 1;
	});

	onMount(() => {
		context = canvas.getContext("2d");

		canvas.width = Math.min(innerWidth, innerHeight);
		canvas.height = Math.min(innerWidth, innerHeight);

		tileSize = innerWidth / 4;

		loop();
	});

	function loop() {
		if (context == null) {
			return;
		}

		context.fillStyle = "black";
		context.fillRect(0, 0, canvas.width, canvas.height);

		for (let i = 0; i < gameGrid.length; i++) {
			let x = (i % tilesPerRow) * tileSize;
			let y = Math.floor(i / tilesPerRow) * tileSize;

			context.fillStyle = "red";
			context.fillRect(x, y, x + tileSize, y + tileSize);
		}

		requestAnimationFrame(loop);
	}
</script>

<canvas bind:this={canvas}></canvas>
