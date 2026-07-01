<script lang="ts">
	import { mount, unmount } from "svelte";
	import PBNotifier from "./PBNotifier.svelte";

	let container: HTMLDivElement;

	let notifiers: PBNotifier[] = [];

	export function newPB(oldTime: number, newTime: number) {
		notifiers.push(
			mount(PBNotifier, {
				target: container,
				props: { oldTime, newTime, onFinished: destroyPB },
			}),
		);
	}

	function destroyPB(pb: PBNotifier) {
		notifiers.splice(
			notifiers.findIndex((el) => el == pb),
			1,
		);
		unmount(pb);
	}
</script>

<div class="container" bind:this={container}>
	{#if false}
		<!-- I'm sorry for trolling you svelte :( -->
		<div></div>
	{/if}
</div>

<style>
	.container {
		position: absolute;
		top: 0;
		bottom: 0;
		width: 100vw;
		height: 100vh;
	}

	.container > * {
		position: absolute;
		bottom: 5vh;
		left: 50vw;
		transform: translate(-50%, 50%);
	}
</style>
