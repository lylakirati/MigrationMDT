<script>
	import { scaleLinear, scaleOrdinal } from 'd3-scale';
	import * as d3 from 'd3';
	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';

	let data = [];
    let dataByMotivation = {
		betterJob:[],
		unemployment:[],
		lackMoneyNeeds: [],
		remittances:[],
		lackMoneyFood: [],
		adventureTourism: [],
	};

	let motivationResponses = {
		1: "betterJob",
		2: "unemployment",
		6: "lackMoneyFood",
		7: "lackMoneyNeeds",
		8: "remittances",
		15: "adventureTourism",
	};

	let dataWithCategories = []

	let order = ["1", "2", "6", "7", "8", "15"]; // to make a 3 x 2 grid

	let categoryNames = {
		1: "Find Better Jobs",
		2: "Unemployment",
		6: "Lack Money for Food",
		7: "Lack Money for Other Basic Needs",
		8: "Send Remittances",
		15: "Tourism"
	}

	let doesNotWantMigrateCounter = 0;
	let wantsMigrateCounter = 0;
	
	// set general use variables
	export let chartWidth = 800;
	let chartHeight = 600;
	let toggle = false;
	export let state = 0; // state of the visualization



	const paddings = {
		top: 25,
		left: 25,
		right: 25,
		bottom: 50
	};
	const padding_between = 20;

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
		.range(['#EFA051', '#1C52A3']);

	// for separating into different types of migration
	$: xScale2 = scaleLinear()
		.domain([0, dotsPerRow/2])
		.range([paddings.left, chartWidth / 2 - padding_between]);
	$: xScale3 = scaleLinear()
		.domain([0, dotsPerRow/2])
		.range([chartWidth / 2 + padding_between, chartWidth - paddings.right]);

	let columnWidth = (chartWidth - paddings.left - paddings.right - 2 * padding_between)/3;
	let categoryDotsPerRow = 36;
	$: xScale4 = scaleLinear() // divide into 3 columns, for different categories
		.domain([0, categoryDotsPerRow])
		.range([0,columnWidth])
	$: colorScaleCategories = scaleOrdinal()
		.domain(order)
		.range(['#4FAA5F', '#1C52A3', '#6297D5', '#824936', '#B990EC', '#F8CE6D']);



	function yScale(index, rowLength) {
		// hard code for now so that we can have fixed distance between dots
		// index: index in the array
		// row length: how many items per row
		return (index - index % rowLength)/rowLength * 9 + paddings.top;
	}

	function xScaleCategories(index, category) {
		let placement = order.indexOf(category); // where it should be placed
		let col = placement % 3;
		let unadjustedX = xScale4(index % (categoryDotsPerRow));
		return unadjustedX + paddings.left + columnWidth*col + padding_between * (col - 1);
	}

	function yScaleCategories(index, category) {
		// place first 3 on first row, second 3 on second row
		let placement = order.indexOf(category); // where it should be placed
		let row = (placement - placement % 3)/3;
		let offset = 300;  // arbitrary offset, find a better way to calculate this
		return yScale(index, categoryDotsPerRow) + row * offset;
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
					.duration(550)
					.ease(d3.easeQuadInOut)
					.attr("cx", (d, i) => xScale(i % dotsPerRow))
            		.attr("cy", (d, i) => yScale(i, dotsPerRow))
            		.attr("r", 3)
					.attr("opacity", 1)
            		.attr("fill", (d) => colorScale(d['mig_ext_intention']))
				break;
			case 2: // second state: split data into people who want to migrate and people who don't want to migrate
				svg
					.selectAll('circle')
					.data(data)
					.join("circle")
					.transition()
					.duration(550)
					.ease(d3.easeQuadInOut)
					.attr('cx', (d) =>
						d['mig_ext_intention'] === 1 ? xScale2(d['ind-2'] % (dotsPerRow/2)) : xScale3(d['ind-2'] % (dotsPerRow/2))
					)
					.attr('cy', (d) => yScale(d['ind-2'], dotsPerRow/2))
            		.attr("r", 3)
					.attr("opacity", 1)
					.attr("fill", (d) => colorScale(d['mig_ext_intention']))
				break;
			case 3: // third case: only look at people who want to migrate by removing all people who don't want to migrate
				svg
					.selectAll('circle')
					.filter((d) => d['mig_ext_intention'] !== 1)
					.transition()
					.duration(550)
					.ease(d3.easeQuadInOut)
					.attr("opacity", 0)
					.remove()

				// svg
				// 	.selectAll('circle')
				// 	.filter((d) => d['mig_ext_intention'] !== 1)
				// 	.transition()
				// 	.duration(550)
				// 	.ease(d3.easeQuadInOut)
				// 	.attr("opacity", 0)
				// 	.remove()

				svg
					.selectAll('circle')
					.data(data.filter(d => d['mig_ext_intention'] === 1))
					.join('circle')
					.transition()
					.delay(550)
					.duration(550)
					.ease(d3.easeQuadInOut)
					.attr("cx", (d, i) => xScale(d['ind-2'] % dotsPerRow))
            		.attr("cy", (d, i) => yScale(d['ind-2'], dotsPerRow))
					.attr("opacity", 1)
					.attr("fill", (d) => colorScale(d['mig_ext_intention']))

				break;
			case 4: 
			case 5: // fourth + fifth case: Separate into different categories based on migration motivation
				svg
					.selectAll('circle')
					.data(dataWithCategories)
					.join('circle')
					.transition()
					.duration(850)
					.ease(d3.easeQuadInOut)
					.attr("cx", (d, i) => xScaleCategories(d.categoryIndex, d.category))
            		.attr("cy", (d, i) => yScaleCategories(d.categoryIndex, d.category))
					.attr("fill", (d) => colorScaleCategories(d.category))
					.attr("opacity", 1)
				break;
			default:
				break;
		}
	}

	$: state, transition();

	onMount(async () => {

		data = await d3.csv("https://raw.githubusercontent.com/lylakirati/MigrationMDT/main/mdt/src/data/main_cleaned.csv");


		data = data.filter((d) => +d['mig_ext_intention'] !== 99)
				.map((d) => ({
					mig_ext_intention: +d['mig_ext_intention'],
					mig_ext_pref_motivo: d['mig_ext_pref_motivo']
				})); // take subset of data for better performance

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
				let motivations = d['mig_ext_pref_motivo'].split(" ");
				for (let motivation of Object.keys(motivationResponses))
				{
					if(motivations.includes(motivation)){
						let curArray = dataByMotivation[motivationResponses[motivation]];
						// todo: refactor to make this more efficient
						dataWithCategories.push({category: motivation, data: {mig_ext_intention: d['mig_ext_intention'], mig_ext_pref_motivo: d['mig_ext_pref_motivo']}, categoryIndex: curArray.length})
						curArray.push(d); // add to relevant data points
					}
					
				}
			}
		}
		
		transition();

		// console.log(doesNotWantMigrateCounter);
		// console.log(wantsMigrateCounter);
		// TODO: populate dataByMotivation to look at the top 5 reasons for migrating

		});
</script>

<section>
	<!-- <div>Current State: {state}</div> -->
	<!-- {#if state === 1}
		<h2>Sorted Migration</h2>
	{:else if state === 2}
		<h2>All People Who Want To Migrate</h2>
	{:else}
		<h2>Unsorted Migration</h2>
	{/if} -->
	<div class="visualization" >
		<!-- {#if state === 1}
			<div class="headers">
				<div>Want To Migrate (Externally)</div>
				<div>Don't Want To Migrate (Externally)</div>
			</div>
		{/if} -->
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
				{#if state === 2}
					<g transition:fade>
						<text 
							x={(chartWidth/2 - padding_between + paddings.left)/2}
							y={((0 - 0 % 2 )/2) * 300 + 15}
							text-anchor={'middle'}
						>
							Wants To Emigrate
						</text>
						<text 
							x={(chartWidth * 3/2 - padding_between + paddings.right)/2}
							y={((1 - 1 %2)/2) * 300 + 15}
							text-anchor={'middle'}>
							Does NOT Want To Emigrate
						</text>
					</g>
				{:else if state === 3}
					<g transition:fade>
						<text 
							x={chartWidth/2}
							y={((0 - 0 % 2 )/2) * 300 + 15}
							text-anchor={'middle'}
						>
							Wants To Emigrate
						</text>
					</g>
				{:else if state === 4 || state === 5}
					<g transition:fade>
						{#each order as category, i}
							<text 
								x={columnWidth * (2 *(i % 3) + 1)/2 + paddings.left + padding_between * (i % 3 - 1)}
								y={((i - i %3)/3) * 300 + 15}
								text-anchor={'middle'}>
								{categoryNames[category]}
							</text>
						{/each}
					</g>
				{/if}
			</svg>
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

  section {
	padding:0;
  }

  .visualization {
	text-align:center;
  }
</style>
