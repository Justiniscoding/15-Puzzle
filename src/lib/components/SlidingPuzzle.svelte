<script lang="ts">
	import { onMount } from "svelte";
	import PBManager from "./PBManager.svelte";

	let { numTiles = $bindable(15) } = $props();

	let canvas: HTMLCanvasElement;
	let context: CanvasRenderingContext2D;
	let pbManager: PBManager;

	let tileSize: number;

	let tilesPerRow = $state(4);

	let blankTileX: number, blankTileY: number;

	let movesDone = $state(0);

	let gameGrid: number[] = [];

	const imagePath = "";
	let imageSize: number;
	let imageTileSize: number;

	let image: HTMLImageElement;

	let startTime = -1;
	let finishedTime = -1;

	type Coord = {
		x: number;
		y: number;
	};

	onMount(() => {
		context = canvas.getContext("2d") ?? new CanvasRenderingContext2D();

		canvas.width = (innerHeight / 3) * 2.1;
		canvas.height = (innerHeight / 3) * 2.1 + 70;

		$effect(() => {
			tilesPerRow = Math.min(Math.max(tilesPerRow, 2), 20);

			numTiles = Math.pow(tilesPerRow, 2);

			blankTileX = tilesPerRow - 1;
			blankTileY = tilesPerRow - 1;

			movesDone = 0;

			if (gameGrid) {
				gameGrid.length = 0;
			}

			gameGrid = new Array(numTiles).fill(null).map((_, index) => {
				if (index == numTiles - 1) {
					return -1;
				}
				return index + 1;
			});

			tileSize = canvas.width / tilesPerRow;

			shuffleGrid();
		});

		window.addEventListener("keydown", swapTilesWithKeyboard);

		if (imagePath === "") {
			loop();
		} else {
			image = new Image();
			image.src = imagePath;

			image.onload = () => {
				imageSize = Math.min(image.width, image.height);
				imageTileSize = imageSize / tilesPerRow;
				loop();
			};
		}
	});

	function loop() {
		context.fillStyle = "black";
		context.fillRect(0, 0, canvas.width, canvas.height);

		context.fillStyle = "#222";
		context.fillRect(0, 0, canvas.width, canvas.width);

		for (let i = 0; i < gameGrid.length; i++) {
			if (gameGrid[i] == -1) {
				continue;
			}

			const x = (i % tilesPerRow) * tileSize;
			const y = Math.floor(i / tilesPerRow) * tileSize;

			const imageX = (gameGrid[i] - 1) % tilesPerRow;
			const imageY = Math.floor((gameGrid[i] - 1) / tilesPerRow);

			if (imagePath == "") {
				const hue = (360 / (numTiles - 1)) * (gameGrid[i] - 1);

				let offset = 0;

				context.fillStyle = `hsl(${hue}, 50%, 50%)`;

				context.fillRect(
					x + offset,
					y + offset,
					tileSize - offset * 2,
					tileSize - offset * 2,
				);

				context.font = "50px Arial";
				context.textBaseline = "middle";
				context.textAlign = "center";

				context.fillStyle = "black";

				context.fillText(
					gameGrid[i].toString(),
					x + tileSize / 2,
					y + tileSize / 2,
				);
			} else {
				context.drawImage(
					image,
					imageX * imageTileSize,
					imageY * imageTileSize,
					imageTileSize,
					imageTileSize,
					x,
					y,
					tileSize,
					tileSize,
				);
			}
		}

		context.fillStyle = "white";

		context.textBaseline = "bottom";
		context.textAlign = "left";

		const textY = canvas.height - 7.5;

		context.fillText(`Moves: ${movesDone}`, 0, textY);

		if (startTime != -1) {
			const timeDifference =
				(finishedTime == -1 ? Date.now() : finishedTime) - startTime;

			const minutes = Math.floor(timeDifference / (1000 * 60));
			const seconds = Math.floor(
				(timeDifference - minutes * 1000 * 60) / 1000,
			).toString();
			const milliseconds = (timeDifference - minutes * 1000 * 60) % 1000;

			let timeString = "";

			if (minutes != 0) {
				timeString += minutes + ":";
			}

			if (seconds.length == 1 && minutes != 0) {
				timeString += "0";
			}

			timeString += seconds + "." + milliseconds;

			context.textAlign = "center";
			context.fillText(`Time: ${timeString}`, canvas.width / 2, textY);

			let movesPerSecond = (movesDone / (timeDifference / 1000)).toFixed(
				1,
			);

			context.textAlign = "right";
			context.fillText(`MPS: ${movesPerSecond}`, canvas.width, textY);
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
		// TODO: Find a more effecient way to do this so that it isn't recalculated every click.
		const canvasOffset = canvas.getBoundingClientRect();

		const tileX = Math.floor((event.x - canvasOffset.left) / tileSize);
		const tileY = Math.floor((event.y - canvasOffset.top) / tileSize);

		let xOffsets = [1, -1, 0, 0];
		let yOffsets = [0, 0, 1, -1];

		for (let i = 0; i < 4; i++) {
			if (
				tileX + xOffsets[i] == blankTileX &&
				tileY + yOffsets[i] == blankTileY
			) {
				if (startTime == -1) {
					startTime = Date.now();
				}

				swapTileWithBlank(tileX, tileY);
				movesDone++;

				if (isSolved(gameGrid)) {
					playerSolvedPuzzle();
				}

				return;
			}
		}
	}

	function swapTilesWithKeyboard(event: KeyboardEvent) {
		let xOffset = 0;
		let yOffset = 0;

		if (event.key == "ArrowLeft") {
			xOffset = 1;
		} else if (event.key == "ArrowRight") {
			xOffset = -1;
		} else if (event.key == "ArrowUp") {
			yOffset = 1;
		} else if (event.key == "ArrowDown") {
			yOffset = -1;
		}

		if (xOffset != 0 || yOffset != 0) {
			if (startTime == -1) {
				startTime = Date.now();
			}

			const tileX = blankTileX + xOffset;
			const tileY = blankTileY + yOffset;

			if (
				tileX < 0 ||
				tileY < 0 ||
				tileX == tilesPerRow ||
				tileY == tilesPerRow
			) {
				return;
			}

			movesDone++;
			swapTileWithBlank(tileX, tileY);

			if (isSolved(gameGrid)) {
				playerSolvedPuzzle();
			}
		}
	}

	async function shuffleGrid() {
		let xOffsets = [1, -1, 0, 0];
		let yOffsets = [0, 0, 1, -1];

		for (let i = 0; i < Math.pow(numTiles, 3); i++) {
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

			// await new Promise((resolve) => setTimeout(resolve, 2));

			swapTileWithBlank(tileX, tileY);
		}
	}

	function solve() {
		if (isSolved(gameGrid)) {
			return;
		}

		type QueueElement = {
			grid: number[];
			blankPosition: Coord;
			lastMove: number;
			moves: Coord[];
		};

		let queue: QueueElement[] = [];

		let explored = new Set<string>();

		queue.push({
			grid: gameGrid,
			blankPosition: { x: blankTileX, y: blankTileY },
			lastMove: -1,
			moves: [],
		});

		const offsetX = [0, -1, 0, 1];
		const offsetY = [1, 0, -1, 0];

		while (queue.length != 0) {
			let element = queue.shift();

			if (element == undefined) {
				continue;
			}

			// console.log(
			// 	`Solver: the queue has ${queue.length} elements at a depth of ${element.moves.length} moves.`,
			// );

			for (let i = 0; i < 4; i++) {
				const gridX = element.blankPosition.x + offsetX[i];
				const gridY = element.blankPosition.y + offsetY[i];

				if (element.lastMove == (i + 2) % 4) {
					continue;
				}

				if (
					gridX < 0 ||
					gridY < 0 ||
					gridX == tilesPerRow ||
					gridY == tilesPerRow
				) {
					continue;
				}

				let newElement: QueueElement = {
					grid: element.grid.slice(),
					blankPosition: { x: 0, y: 0 },
					lastMove: i,
					moves: element.moves.slice(),
				};

				const tileIndex = gridY * tilesPerRow + gridX;
				const blankIndex =
					element.blankPosition.y * tilesPerRow +
					element.blankPosition.x;

				newElement.blankPosition.x = gridX;
				newElement.blankPosition.y = gridY;

				const temp = newElement.grid[tileIndex];

				newElement.grid[tileIndex] = newElement.grid[blankIndex];
				newElement.grid[blankIndex] = temp;

				const gridAsString = newElement.grid.join("");

				if (!explored.has(gridAsString)) {
					explored.add(gridAsString);

					newElement.moves.push({ x: gridX, y: gridY });

					if (isSolved(newElement.grid)) {
						console.log(
							`Solver: Found a ${newElement.moves.length} move solution.`,
						);

						for (let i = 0; i < newElement.moves.length; i++) {
							setTimeout(() => {
								movesDone++;
								swapTileWithBlank(
									newElement.moves[i].x,
									newElement.moves[i].y,
								);
							}, i * 300);
						}

						queue.length = 0;

						break;
					}

					queue.push(newElement);
				}
			}
		}
	}

	async function algorithmicSolve() {
		for (
			let targetCell = 0;
			targetCell < tilesPerRow * tilesPerRow - tilesPerRow * 2;
			targetCell++
		) {
			let currentTarget = targetCell + 1;

			if (currentTarget % tilesPerRow == tilesPerRow - 1) {
				currentTarget++;
			} else if (currentTarget % tilesPerRow == 0) {
				currentTarget--;
			}

			let targetCurrentPosition: Coord = { x: -1, y: -1 };
			let targetHomePosition: Coord = {
				x: targetCell % tilesPerRow,
				y: Math.floor(targetCell / tilesPerRow),
			};

			if (currentTarget % tilesPerRow == tilesPerRow - 1) {
				targetHomePosition.x--;
				targetHomePosition.y++;
			}

			for (let i = 0; i < gameGrid.length; i++) {
				if (gameGrid[i] == currentTarget) {
					targetCurrentPosition = {
						x: i % tilesPerRow,
						y: Math.floor(i / tilesPerRow),
					};
				}
			}

			if (
				targetCurrentPosition.x == targetHomePosition.x &&
				targetCurrentPosition.y == targetHomePosition.y
			) {
				continue;
			}

			// Bring the blank position to the target
			const manhattanDistance =
				Math.abs(targetCurrentPosition.x - blankTileX) +
				Math.abs(targetCurrentPosition.y - blankTileY);

			for (let i = 0; i < manhattanDistance - 1; i++) {
				if (targetCurrentPosition.x != blankTileX) {
					swapTileWithBlank(
						Math.sign(targetCurrentPosition.x - blankTileX) +
							blankTileX,
						blankTileY,
					);
					movesDone++;
				} else {
					swapTileWithBlank(
						blankTileX,
						Math.sign(targetCurrentPosition.y - blankTileY) +
							blankTileY,
					);
					movesDone++;
				}

				// await new Promise((resolve) => setTimeout(resolve, 300));
			}

			// Swap target with blank if it brings the target closer to its home position
			if (
				Math.abs(blankTileX - targetHomePosition.x) <
					Math.abs(targetHomePosition.x - targetCurrentPosition.x) ||
				Math.abs(blankTileY - targetHomePosition.y) <
					Math.abs(targetHomePosition.y - targetCurrentPosition.y)
			) {
				const oldBlankX = blankTileX;
				const oldBlankY = blankTileY;

				swapTileWithBlank(
					targetCurrentPosition.x,
					targetCurrentPosition.y,
				);
				movesDone++;

				targetCurrentPosition.x = oldBlankX;
				targetCurrentPosition.y = oldBlankY;

				// await new Promise((resolve) => setTimeout(resolve, 1000));
			}

			// Bring the target to its position
			while (
				targetCurrentPosition.x != targetHomePosition.x ||
				targetCurrentPosition.y != targetHomePosition.y
			) {
				// Move target to the right until it is aligned with home
				if (targetCurrentPosition.x < targetHomePosition.x) {
					let xOffsets: number[] = [1, 0, -1, 0, 1];
					let yOffsets: number[] = [0, -1, 0, 1, 0];

					const isXAligned = blankTileX == targetCurrentPosition.x;

					for (let i = 0; i < xOffsets.length; i++) {
						if (!isXAligned && i == 0) {
							// Add new moves to set up for the swapping.
							swapTileWithBlank(blankTileX, blankTileY + 1);
							// await new Promise((resolve) =>
							// 	setTimeout(resolve, 300),
							// );
							swapTileWithBlank(blankTileX + 1, blankTileY);
							// await new Promise((resolve) =>
							// 	setTimeout(resolve, 300),
							// );
						}

						swapTileWithBlank(
							blankTileX + xOffsets[i],
							blankTileY + yOffsets[i],
						);

						movesDone++;
						// await new Promise((resolve) =>
						// 	setTimeout(resolve, 100),
						// );
					}

					targetCurrentPosition.x++;
				} else if (targetCurrentPosition.y != targetHomePosition.y) {
					// Move target up until it is aligned with home
					let xDirection = 1;

					if (targetCurrentPosition.x == tilesPerRow - 1) {
						xDirection = -1;
					}

					let xOffsets: number[] = [xDirection, 0, 0, -xDirection, 0];
					let yOffsets: number[] = [0, -1, -1, 0, 1];

					const isXAligned = blankTileX == targetCurrentPosition.x;

					for (let i = 0; i < xOffsets.length; i++) {
						// Skip first two movements if blank is to the left/right of target current position.
						if (!isXAligned && i < 2) {
							continue;
						}

						swapTileWithBlank(
							blankTileX + xOffsets[i],
							blankTileY + yOffsets[i],
						);
						movesDone++;

						// await new Promise((resolve) =>
						// 	setTimeout(resolve, 100),
						// );
					}

					targetCurrentPosition.y--;
				} else {
					// Move target to the left until it is aligned with home
					let xOffsets: number[] = [-1, 0, 1, 0, -1];
					let yOffsets: number[] = [0, -1, 0, 1, 0];

					const isXAligned = blankTileX == targetCurrentPosition.x;

					for (let i = 0; i < xOffsets.length; i++) {
						if (!isXAligned && i == 0) {
							// Add new moves to set up for the swapping.
							swapTileWithBlank(blankTileX, blankTileY + 1);
							// await new Promise((resolve) =>
							// 	setTimeout(resolve, 300),
							// );
							swapTileWithBlank(blankTileX - 1, blankTileY);
							// await new Promise((resolve) =>
							// 	setTimeout(resolve, 300),
							// );
						}

						swapTileWithBlank(
							blankTileX + xOffsets[i],
							blankTileY + yOffsets[i],
						);
						movesDone++;

						// await new Promise((resolve) =>
						// 	setTimeout(resolve, 100),
						// );
					}

					targetCurrentPosition.x--;
				}
			}

			if (currentTarget % tilesPerRow == tilesPerRow - 1) {
				let xOffsets: number[] = [1, 0, 0, -1, 0];
				let yOffsets: number[] = [0, -1, -1, 0, 1];

				for (let i = 0; i < xOffsets.length; i++) {
					swapTileWithBlank(
						blankTileX + xOffsets[i],
						blankTileY + yOffsets[i],
					);
					movesDone++;

					// await new Promise((resolve) => setTimeout(resolve, 100));
				}
			}
		}
	}

	function isSolved(gameGrid: number[]) {
		if (gameGrid[gameGrid.length - 1] != -1) {
			return false;
		}

		for (let i = 1; i < gameGrid.length - 1; i++) {
			if (gameGrid[i - 1] > gameGrid[i]) {
				return false;
			}
		}

		return true;
	}

	function resetGame() {
		shuffleGrid();
		movesDone = 0;
		startTime = -1;
		finishedTime = -1;
	}

	function testSpeed() {
		const startTime = Date.now();

		for (let i = 0; i < 500; i++) {
			shuffleGrid();
			solve();
		}

		const endTime = Date.now();

		const timeTaken = endTime - startTime;
		const secondsTaken = Math.floor(timeTaken / 1000);
		let millisecondsTaken = (timeTaken % 1000).toString();

		while (millisecondsTaken.length != 3) {
			millisecondsTaken = "0" + millisecondsTaken;
		}

		console.log(`It took ${secondsTaken}:${millisecondsTaken} seconds`);
	}

	function playerSolvedPuzzle() {
		finishedTime = Date.now();

		const timeTaken = finishedTime - startTime;
		const localStorageKey = `${numTiles - 1}_puzzle_pb`;

		if (
			localStorage[localStorageKey] &&
			timeTaken < localStorage[localStorageKey]
		) {
			pbManager.newPB(
				localStorage[localStorageKey],
				timeTaken,
				numTiles - 1,
			);
			localStorage[localStorageKey] = timeTaken;
		}
		if (!localStorage[localStorageKey]) {
			localStorage[localStorageKey] = timeTaken;
		}
	}
</script>

<canvas bind:this={canvas} onclick={swapTilesOnClick}></canvas>

<!-- <button onclick={solve}>Solve</button> -->
<!-- <button onclick={algorithmicSolve}>Algorithmic Solve (not optimal)</button> -->
<!-- <button onclick={testSpeed}>Performance test</button> -->
<div class="bottomContainer">
	<div class="puzzleSizeContainer">
		<label for="tilesPerRow">Puzzle Size</label>
		<input
			type="number"
			id="tilesPerRow"
			min="2"
			max="20"
			bind:value={tilesPerRow}
		/>
	</div>
	<button onclick={resetGame}>Restart</button>
</div>

<PBManager bind:this={pbManager}></PBManager>

<style>
	canvas {
		border: 5px solid black;
		border-radius: 4px;
	}

	:global(button) {
		padding: 1vw 1.5vw;
		border: 2px solid black;
		border-radius: 0.5vw;
		margin-top: 2vh;
		font-size: 2vh;
		background-color: #1e90ff;
		cursor: pointer;
	}

	label {
		font-size: min(3vw, 3vh);
		font-family: Arial, Helvetica, sans-serif;
	}

	input {
		width: min(4vw, 4vh);
		font-size: min(3vw, 3vh);
		border: 2px solid black;
		border-radius: 0.5vw;
	}

	.bottomContainer {
		margin-top: min(3vw, 3vh);
		display: flex;
		justify-content: space-evenly;
		align-items: center;
		width: 90vw;
	}

	.bottomContainer > button {
		margin: 0;
	}

	.puzzleSizeContainer {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
</style>
