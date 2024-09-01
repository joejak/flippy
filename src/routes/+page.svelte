<script>
	import Flip from '$lib/Flip.svelte';
	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';
	let editor;
	let fadeScaler = 10;
	let previewScaler = .15;
	/**
	 * @type {HTMLDivElement}
	 */
	let sideScroller;
	let stack = writable([]);
	let cdim = writable({
		w: 400,
		h: 200
	});
	/**
	 * @type {HTMLDivElement}
	 */
	let snaps;
	onMount(() => {
		$cdim.w = (window.visualViewport?.width ?? 400) * 0.7;
		$cdim.h = (window.visualViewport?.height ?? 200) * 0.7;
	});

	/**
	 * @param {CustomEvent<string>} data
	 */
	function save(data) {
		let frame = createFrame(data.detail);
		$stack.unshift(frame);
		$stack = $stack;
		sideScroller.scrollLeft = sideScroller.scrollWidth;
	}

	/**
	 * @param {string} data
	 */
	function createFrame(data) {
		const img = document.createElement('img');
		img.src = data;
		return img;
	}
</script>

<div style="display:flex; flex-direction: column; width:{$cdim.w+2}px; padding:1px" bind:this={editor}>
	<div style="border:solid; position:relative; ">
		{#each $stack as frame, index}
			<img
				style="position:absolute; opacity:{1 / ((index + 1) * fadeScaler)}; z-index:-1;"
				src={frame.src}
				alt="the last completed flip page"
			/>
		{/each}
		<Flip w={$cdim.w} h={$cdim.h} on:save={(data) => save(data)}></Flip>
	</div>
	<div bind:this={sideScroller} style="display: flex; flex-direction: row-reverse; width:auto; overflow-x: scroll; border:solid; height: {$cdim.h*previewScaler}px; padding-right: {($cdim.w/2)-$cdim.w*(previewScaler/2)}px">
		{#each $stack as frame, index}
			<img src={frame.src} alt="image {index} in the reel" style="border: solid grey;"/>
		{/each}
	</div>
</div>
