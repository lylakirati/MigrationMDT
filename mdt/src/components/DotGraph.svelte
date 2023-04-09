<script>
	import { scaleLinear } from 'd3-scale';
	import * as d3 from 'd3';

	import { onMount } from 'svelte';

	let data = [];

	let doesNotWantMigrateCounter = 0;
	let wantsMigrateCounter = 0;

	onMount(async () => {
		data = await d3.csv('../../src/data/main_cleaned.csv');

		data = data.filter((d) => +d['mig_ext_intention'] !== 99);
		for (let d of data) {
			d['mig_ext_intention'] = +d['mig_ext_intention'];
			if (d['mig_ext_intention'] === 0) {
				// set the second index position to the not migrating counter
				d['ind-2'] = doesNotWantMigrateCounter;
				doesNotWantMigrateCounter += 1;
			} else {
				// set the second index position to the migrating counter
				d['ind-2'] = wantsMigrateCounter;
				wantsMigrateCounter += 1;
			}
		}
		console.log(doesNotWantMigrateCounter, wantsMigrateCounter);
	});

	// set general use variables
	let chartWidth = 720;
	let chartHeight = 1800;

	const paddings = {
		top: 25,
		left: 50,
		right: 50,
		bottom: 50
	};
	const padding_between = 10;

	const dotsPerRow = 50;

	// set scaling variables
	$: xScale = scaleLinear()
		.domain([0, dotsPerRow])
		.range([paddings.left, chartWidth - paddings.right / 2]);
	$: yScale = scaleLinear()
		.domain([0, (data.length - (data.length % dotsPerRow)) / dotsPerRow])
		.range([paddings.bottom, chartHeight - paddings.top]);

	$: xScale2 = scaleLinear()
		.domain([0, 25])
		.range([paddings.left, chartWidth / 2 - padding_between]);
	$: xScale3 = scaleLinear()
		.domain([0, 25])
		.range([chartWidth / 2 + padding_between, chartWidth - paddings.right]);

	let toggle = false;
	let state = 0;
	function transition() {
		if (state === 0) {
			d3.select('svg')
				.selectAll('circle')
				.data(data)
				.join('circle')
				.transition()
				.duration(250)
				.ease(d3.easeLinear)
				.attr('cx', (d) =>
					d['mig_ext_intention'] === 1 ? xScale2(d['ind-2'] % 25) : xScale3(d['ind-2'] % 25)
				)
				.attr('cy', (d) => yScale((d['ind-2'] - (d['ind-2'] % 25)) / 25));
			state = 1;
		} else if (state === 1) {
			d3.select('svg')
				.selectAll('circle')
				.data(data.filter((d) => d['mig_ext_intention'] === 1))
				.join('circle')
				.join(
					function (enter) {
						return enter.append('circle');
					},
					function (update) {
						return update;
					},
					function (exit) {
						return exit.transition().attr('cy', 500).remove();
					}
				)
				.transition()
				// .duration(250)
				// .ease(d3.easeLinear)
				// .attr('cx', (d, i) => xScale(i % dotsPerRow))
				// .attr('cy', (d, i) => yScale((i - (i % dotsPerRow)) / dotsPerRow));
			state = 2;
		} else if (state === 2) {
			d3.select('svg')
				.selectAll('circle')
				.data(data)
				.join('circle')
				.transition()
				.duration(250)
				.ease(d3.easeLinear)
				.attr('cx', (d, i) => xScale(i % dotsPerRow))
				.attr('cy', (d, i) => yScale((i - (i % dotsPerRow)) / dotsPerRow));
			state = 0;
		}
		toggle = !toggle; // to allow toggling back
	}
</script>

<main>
	<label>
		<button on:click={() => transition()}> Current State: {state} </button>
	</label>
	{#if state === 0}
		<h2>Sorted Migration</h2>
	{:else}
		<h2>Unsorted Migration</h2>
	{/if}
	<div class="visualization">
		{#if state === 1}
			<div class="headers">
				<div>Want To Migrate (Externally)</div>
				<div>Don't Want To Migrate (Externally)</div>
			</div>
		{/if}
		{#if data.length > 1}
			<svg width={chartWidth} height={chartHeight}>
				<g>
					{#each data as d, i}
						{#if i != data.length - 1}
							<circle
								cx={xScale(i % 50)}
								cy={yScale((i - (i % 50)) / 50)}
								id={`dot-${i}`}
								r={3}
								fill={+d['mig_ext_intention'] === 0 ? '#65BABD' : '#000'}
							/>
						{/if}
					{/each}
				</g>
			</svg>
			<!-- <div
                class={mousePosition.x === null
                    ? "tooltip-hidden"
                    : "tooltip-visible"}
                style="left: {pageMousePosition.x +
                    10}px; top: {pageMousePosition.y + 10}px"
            >
                {#if mousePosition.x !== null}
                    At index {currentHoveredPoint.index}, the size was {currentHoveredPoint.size}.
                {/if}
            </div> -->
		{:else}
			<p>Loading</p>
		{/if}
	</div>
</main>

<style>
	main {
		text-align: center;
		font-family: 'Nunito', sans-serif;
		font-weight: 300;
		line-height: 1;
		/* font-size: 12px; */
		color: var(--color-text);
		margin-top: 100px;
	}

	@keyframes draw {
		from {
			stroke-dashoffset: 4400;
		}
		to {
			stroke-dashoffset: 0;
		}
	}
	.headers {
		display: flex;
		flex-direction: row;
		justify-content: center;
		gap: 50px;
	}
</style>
