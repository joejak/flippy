<script>
	import Flip from '$lib/Flip.svelte';
	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';
	let cdim = writable({
		w: 400,
		h: 200
	});
  /**
	 * @type {HTMLDivElement}
	 */
  let snaps; 
	onMount(() => {
		$cdim.w = (window.visualViewport?.width ?? 400)*.9;
		$cdim.h = (window.visualViewport?.height ?? 200)*.9;
	});

 /**
	 * @param {CustomEvent<string>} data
	 */
 function save(data){
  const img = document.createElement('img'); 
  img.src = data.detail; 
  snaps.prepend(img);
 }
</script>

<Flip w={$cdim.w} h={$cdim.h} on:save={(data) => save(data)}></Flip>

<div bind:this={snaps}></div>
