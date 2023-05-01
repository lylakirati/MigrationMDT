<script>
	import { scaleLinear, scaleOrdinal } from 'd3-scale';
	import * as d3 from 'd3';

	import { onMount } from 'svelte';

	let data = [];
    let dataByMotivation = {
		betterJob:[],
		unemployment:[],
		lackMoneyNeeds: [],
		remittances:[],
		lackMoneyFood: [],
	};

	let doesNotWantMigrateCounter = 0;
	let wantsMigrateCounter = 0;
	
	// set general use variables
	export let chartWidth = 850;
	let chartHeight = 600;
	let toggle = false;
	export let state = 0; // state of the visualization



	const paddings = {
		top: 10,
		left: 25,
		right: 25,
		bottom: 50
	};
	const padding_between = 10;

	const dotsPerRow = 100;

	// set scaling variables
	$: xScale = scaleLinear()
		.domain([0, dotsPerRow])
		.range([paddings.left, chartWidth - paddings.right / 2]);
	// $: yScale = scaleLinear()
	// 	.domain([0, (data.length - (data.length % dotsPerRow)) / dotsPerRow])
	// 	.range([paddings.top, chartHeight - paddings.bottom]);
    $: colorScale = scaleOrdinal()
		.domain([0, 1])
		.range(['#000', '#65BABD']);

	// for separating into different types of migration
	$: xScale2 = scaleLinear()
		.domain([0, dotsPerRow/2])
		.range([paddings.left, chartWidth / 2 - padding_between]);
	$: xScale3 = scaleLinear()
		.domain([0, dotsPerRow/2])
		.range([chartWidth / 2 + padding_between, chartWidth - paddings.right]);

	$: xScale4 = scaleLinear() // divide into 3 columns, for different categories
		.domain([0, dotsPerRow/3])
		.range([0, (chartWidth - paddings.left - paddings.right - 2 * padding_between)/3])

	function yScale(index, rowLength) {
		// hard code for now so that we can have fixed distance between dots
		// index: index in the array
		// row length: how many items per row
		return (index - index % rowLength)/rowLength * 9 + paddings.top;
	}

	function xScaleCategories(index, category) {
		
	}

	function yScaleCategories(index, category) {
		// place first 3 on first row, second 3 on second row
	}

	// TODO: implement this, then move this to case 2
	// 			// This should sorta be structured as the following: (at least my 2 AM brain thinks it should be)
	// 			// 1. populate dataByMotivations arrays
	// 			// 2. Create an XScale for them (this can probably be the same xScale that is then transformed for
	// 			//    each thing) --> draw a sketch of this somewhere so you can figure out the right scalings
	// 			// 3. Do we want a YScale, or just define what the gap should be between each one? If we do that, we 
	// 			//    should do that for the previous scales as well
	// 			// 4. Assign colors based on our color palette
	// 			// 5. Figure out Transitions
	// 			// 6. Figure out how to make this entire visualization fit in the screen in the first place
	function transition() {
		// take 2 because i think the first one is doomed
		if(data.length === 0)
		{
			return; // do nothing if there's no data yet
		}
		console.log("Current state", state)
		let svg = d3.select('svg').select('g');
		switch(state) {
			case 0: // first state: all the data displayed
				svg
					.selectAll('circle')
					.data(data)
					.join(
						enter => enter.append("circle")
					)
					.transition()
					.duration(250)
					.ease(d3.easeLinear)
					.attr("cx", (d, i) => xScale(i % dotsPerRow))
            		.attr("cy", (d, i) => yScale(i, dotsPerRow))
            		.attr("r", 3)
            		.attr("fill", (d) => colorScale(d['mig_ext_intention']))
				break;
			case 1: // second case: split data into people who want to migrate and people who don't want to migrate
				svg
					.selectAll('circle')
					.data(data)
					.join("circle")
					.transition()
					.duration(250)
					.ease(d3.easeLinear)
					.attr('cx', (d) =>
						d['mig_ext_intention'] === 1 ? xScale2(d['ind-2'] % (dotsPerRow/2)) : xScale3(d['ind-2'] % (dotsPerRow/2))
					)
					.attr('cy', (d) => yScale(d['ind-2'], dotsPerRow/2))
            		.attr("r", 3)
					.attr("fill", (d) => colorScale(d['mig_ext_intention']))
				break;
			case 2: // third case: only look at people who want to migrate by removing all people who don't want to migrate
				svg
					.selectAll('circle')
					.data(data.filter(d => d['mig_ext_intention'] === 1))
					.join('circle')
					.transition()
					.duration(250)
					.ease(d3.easeLinear)
					.attr("cx", (d, i) => xScale(i % dotsPerRow))
            		.attr("cy", (d, i) => yScale(i, dotsPerRow))
					.attr("fill", (d) => colorScale(d['mig_ext_intention']))
			default:
				break;
		}
	}

	$: state, transition();

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
		state = 1;
		state = 0;
		transition();
		// TODO: populate dataByMotivation to look at the top 5 reasons for migrating

		});
//     // hover effect
//   const idContainer = "svg-container-" + Math.random() * 1000000;
//   let mousePosition = { x: null, y: null };
//   let pageMousePosition = { x: null, y: null };
//   let currentHoveredPoint = null;

//   function followMouse(event) {
//     const svg = document.getElementById(idContainer);
//     if (svg === null) return;
//     const dim = svg.getBoundingClientRect();
//     pageMousePosition = {
//       x: event.pageX,
//       y: event.pageY,
//     };
//     const positionInSVG = {
//       x: event.clientX - dim.left,
//       y: event.clientY - dim.top,
//     };
//     mousePosition =
//       positionInSVG.x > paddings.left &&
//       positionInSVG.x < chartWidth - paddings.right &&
//       positionInSVG.y > paddings.top &&
//       positionInSVG.y < chartHeight - paddings.bottom
//         ? { x: positionInSVG.x, y: positionInSVG.y }
//         : { x: null, y: null };
//     computeSelectedXYValue(mousePosition.x, mousePosition.y);
//   }
//   function removePointer() {
//     mousePosition = { x: null, y: null };
//   }
//     function computeSelectedXYValue(xVal, yVal) {
//         currentHoveredPoint =
//             data.filter((d, i) => xScale(i % dotsPerRow) >= xVal && yScale((i - i % dotsPerRow)/dotsPerRow) >= yVal)[0];
//         console.log(xVal, yVal, currentHoveredPoint);
//         return null;
//     }
</script>

<section>
	<div>Current State: {state}</div>
	{#if state === 1}
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
			<svg width={chartWidth} height={chartHeight} >
				<g></g>
				<!-- {#each data as d, i}
					{#if i != data.length - 1}
						<circle
							cx={xScale(i % dotsPerRow)}
							cy={yScale((i - (i % dotsPerRow)) / dotsPerRow)}
							id={`dot-${i}`}
							r={3}
							fill={colorScale(+d['mig_ext_intention'])}
						/>
					{/if}
				{/each} -->
			</svg>
			<!-- <div
                class={mousePosition.x === null
                    ? "tooltip-hidden"
                    : "tooltip-visible"}
                style="left: {pageMousePosition.x +
                    10}px; top: {pageMousePosition.y + 10}px"
            >
                {#if mousePosition.x !== null}
                    Migration Motives: {currentHoveredPoint['mig_ext_pref_motivo']}.
                {/if}
            </div> -->
	</div>
</section>

<style>

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

        /* dynamic classes for the tooltip */
  .tooltip-hidden {
    visibility: hidden;
    font-family: "Nunito", sans-serif;
    width: 200px;
    position: absolute;
  }

  .tooltip-visible {
    font: 25px sans-serif;
    font-family: "Nunito", sans-serif;
    visibility: visible;
    background-color: #f0dba8;
    border-radius: 10px;
    width: 200px;
    color: black;
    position: absolute;
    padding: 10px;
  }
</style>
