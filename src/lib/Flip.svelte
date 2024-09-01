<script>
	import { createEventDispatcher, onMount } from 'svelte';

	export let w = 900;
	export let h = 600;

	export const event = createEventDispatcher();

	/**
	 * @type HTMLCanvasElement
	 */
	let canvas;

	/**
	 * @type CanvasRenderingContext2D | null
	 */
	let ctx;

	let drawing = false;
	/**
	 * @type {{ x: number; y: number; }[][]}
	 */
	let pathsry = [];
	/**
	 * @type {{ x: number; y: number; }[]}
	 */
	let points = [];

	var mouse = { x: 0, y: 0 };
	var previous = { x: 0, y: 0 };

	onMount(() => {
		ctx = canvas.getContext('2d');
		if (ctx) {
			ctx.lineJoin = 'round';
			ctx.lineCap = 'round';
			ctx.strokeStyle = 'black';

			canvas.addEventListener(
				'mousemove',
				function (e) {
					if (drawing) {
						previous = { x: mouse.x, y: mouse.y };
						mouse = oMousePos(e);
						// saving the points in the points array
						points.push({ x: mouse.x, y: mouse.y });
						// drawing a line from the previous point to the current point
						ctx?.beginPath();
						ctx?.moveTo(previous.x, previous.y);
						ctx?.lineTo(mouse.x, mouse.y);
						ctx?.stroke();
					}
				},
				false
			);

			canvas.addEventListener(
				'mouseup',
				function () {
					drawing = false;
					// Adding the path to the array or the paths
					pathsry.push(points);
				},
				false
			);

			canvas.addEventListener('mousedown', function (e) {
				drawing = true;
				previous = { x: mouse.x, y: mouse.y };
				mouse = oMousePos(e);
				points = [];
				points.push({ x: mouse.x, y: mouse.y });
			});
		} else {
			console.error('failed to load ctx!!');
		}
	});

	function drawPaths() {
		if (ctx) {
			// delete everything
			clearCanvas();
			// draw all the paths in the paths array
			pathsry.forEach((path) => {
				if (ctx) {
					ctx.beginPath();
					ctx.moveTo(path[0].x, path[0].y);
					for (let i = 1; i < path.length; i++) {
						ctx.lineTo(path[i].x, path[i].y);
					}
					ctx.stroke();
				}
			});
		}
	}

	function clearCanvas() {
		if (ctx) {
			ctx.clearRect(0, 0, canvas.width, canvas.height);
		}
	}

	function undo() {
		// remove the last path from the paths array
		pathsry.splice(-1, 1);
		// draw all the paths in the paths array
		drawPaths();
	}

	/**
	 * @param {MouseEvent} evt
	 */
	function oMousePos(evt) {
		var ClientRect = canvas.getBoundingClientRect();
		return {
			//objeto
			x: Math.round(evt.clientX - ClientRect.left),
			y: Math.round(evt.clientY - ClientRect.top)
		};
	}

	function save() {
		event('save', canvas.toDataURL(), { cancelable: true });
	}
</script>

<div>
	<div class="ctrls">
		<button on:click={() => save()}>save</button>
		<button on:click={() => undo()}>undo</button>
		<button on:click={() => clearCanvas()}>clear</button>
	</div>
	<canvas style="border:solid; margin-top:20px" bind:this={canvas} width={w} height={h}> </canvas>
</div>

<style>
	.ctrls {
		position: absolute;
		right: 10px;
		opacity: 0.2;
		z-index: 99;
	}
	.ctrls:hover {
		opacity: 1;
	}
</style>
