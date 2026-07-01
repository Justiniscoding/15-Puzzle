<script lang="ts">
	import { mount, unmount } from "svelte";
	import PBNotifier from "./PBNotifier.svelte";

	let container: HTMLDivElement;

	let notifiers: Map<number, PBNotifier> = new Map();

	let nextMapIndex = 0;

	export function newPB(
		oldTime: number,
		newTime: number,
		puzzleSize: number,
	) {
		notifiers.set(
			nextMapIndex,
			mount(PBNotifier, {
				target: container,
				props: {
					oldTime,
					newTime,
					puzzleSize,
					onFinished: destroyPB,
					index: nextMapIndex++,
				},
			}),
		);
	}

	function destroyPB(mapIndex: number) {
		unmount(notifiers.get(mapIndex)!);
		notifiers.delete(mapIndex);
	}
</script>

<div class="container" bind:this={container}></div>
