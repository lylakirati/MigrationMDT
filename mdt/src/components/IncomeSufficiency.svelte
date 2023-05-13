<script>
	import { scaleLinear, scaleOrdinal } from 'd3-scale';
	// import { readFileSync } from 'fs';
	// import path from 'path';
	// import { promises as fs } from 'fs'
	// import { dsvFormat } from 'd3'
	import * as d3 from 'd3';

	import { onMount } from 'svelte';
	import {fade} from 'svelte/transition';

	let countryMap = {
		SLV: 'El Salvador',
		GT: 'Guatemala',
		HND: 'Honduras'
	}
	
	let data = [];
	let incomeSufficiency = {
		enough: [],
		almostEnough: [],
		sometimesEnough: [],
		rarelyEnough: [],
		insufficient: []
	};

	let incomeSufficiencyAmounts = {
		enough: 0,
		almostEnough: 0,
		sometimesEnough: 0,
		rarelyEnough: 0,
		insufficient: 0
	};
	
	// for translating between values and their actual display names
	let incomeSufficiencyKeyMap = {
		enough: "Enough",
		almostEnough: "Almost Enough",
		sometimesEnough: "Sometimes Enough",
		rarelyEnough: "Rarely Enough",
		insufficient: "Insufficient"
	};
	// map survey responses to income sufficiency levels
	let incomeSufficiencyAssocs = {
		1: 'enough',
		2: 'almostEnough',
		3: 'sometimesEnough',
		4: 'rarelyEnough',
		5: 'insufficient'
	};
	let maxVal = 0;

	// load data and partition into income sufficiency values
	onMount(async () => {
		// data = await d3.csv('src/data/main_cleaned.csv');
		
		data = await d3.csv("https://raw.githubusercontent.com/lylakirati/MigrationMDT/main/mdt/src/data/main_cleaned.csv");
		// https://raw.githubusercontent.com/lylakirati/MigrationMDT/main/mdt/src/data/main_cleaned.csv
		data = data.filter((d) => +d['income_sufficiency_6m'] !== 99);

		// const file = path.join(process.cwd(), 'src', 'data/main_cleaned.csv');
  		// data = readFileSync(file, 'utf8');
		// data = data.filter((d) => +d['income_sufficiency_6m'] !== 99);

		// const filePath = path.join(process.cwd(), 'src/data/main_cleaned.csv');
  		// const fileString = await fs.readFile(filePath, 'utf8');
		// data = csv.parse(fileString, (d) => {
		// 	if (d.slug === context.params.id) return d;
		// })

		for (let d of data) {
			incomeSufficiency[incomeSufficiencyAssocs[+d['income_sufficiency_6m']]].push(d);
		}
		for (const key of Object.keys(incomeSufficiencyAmounts)) {
			incomeSufficiencyAmounts[key] = incomeSufficiency[key].length;
		}
		const amounts = Object.keys(incomeSufficiencyAmounts).map(
			(key) => incomeSufficiencyAmounts[key]
		);
		maxVal = Math.max(...amounts);
	});

	let countryToggle = false;
	let curCountry = '';

	let buttons = {
		hnd: undefined,
		slv: undefined,
		gt: undefined
	};

	// update income sufficiency data with new data (without changing outside data variable)
	function updateData(data) {
		for (const key of Object.keys(incomeSufficiency))
		{
			incomeSufficiency[key] = [];
		}
		for (let d of data) {
			incomeSufficiency[incomeSufficiencyAssocs[+d['income_sufficiency_6m']]].push(d);
		}
		// console.log(incomeSufficiency);
		for (const key of Object.keys(incomeSufficiencyAmounts)) {
			incomeSufficiencyAmounts[key] = incomeSufficiency[key].length;
		}
	}

	
	// update country, as well as makes it so that the button that was pressed is toggled
	function updateCountry(country) {
		// var selectBox = document.getElementById("select-country");
		// var selectedCountry = selectBox.options[selectBox.selectedIndex].value;
		// alert(selectedValue);
		if (country === 'all' || (country === curCountry && countryToggle)) {
			// if equal and country active, reset
			updateData(data);
			countryToggle = false;
			curCountry = '';

			d3.selectAll("button")
				.classed("active", false);

			d3.select("#button-all")
				.classed("active", true);

			return;
		}

		d3.selectAll("button")
			.classed("active", false);

		d3.select("#button-" + country)
			.classed("active", true);
		
		countryToggle = true;
		curCountry = country;
		updateData(data.filter((d) => d.country === country));
	}
	


	// overall chart width, height, and paddings
	let chartWidth = 700; // 800
	let chartHeight = 1400; // 1000
	const paddings = {
		top: 25,
		left: 25,
		right: 25,
		bottom: 50
	};
	const paddingBetween = 25; // padding between each column

	// individual column width
	let numCategories = 5;
	let columnWidth =
		(chartWidth - paddings.right - paddings.left - 4 * paddingBetween) / numCategories;

	const dotsPerRow = 12; // 15

	// set scaling variables
	$: xScale = scaleLinear()
		.domain([0, dotsPerRow - 1])
		.range([0, columnWidth]); // do positioning of svg's after xScale calculated
	$: yScale = scaleLinear() // everything scales full height
		.domain([0, (maxVal - (maxVal % dotsPerRow)) / dotsPerRow])
		.range([paddings.bottom, chartHeight - paddings.top]);
	$: colorScale = scaleOrdinal()
		.domain([0, 1, 2, 3, 4])
		.range(['#CCF3FF', '#B9E1FF', '#629FD5', '#1C52A3', '#0E244F']);
	
		

</script>

<main>
	
	<label class="selections">
		Choose a country:
		<button id = "button-all" class = "active" on:click={() => updateCountry('all')}> All Countries </button>
		{#each Object.keys(countryMap) as country}
			<button id = "button-{country}" on:click={() => updateCountry(country)}> {countryMap[country]} </button>
		{/each}
	</label>
	

	<h2 class={"current-country"}>Levels of income sufficiency for households in {countryToggle ? countryMap[curCountry] : 'all three countries'}</h2>

	<div>One <svg width={10} height={10}><circle r={3} cx={5} cy={5}></circle></svg>  represents 5 households</div>
	<div class="visualization">
		{#if data.length > 1}
			<svg width={chartWidth} height={chartHeight}>
				{#each Object.keys(incomeSufficiencyAmounts) as key, ind}
					<g
						class={key}
						transform={`translate(${paddings.left + ind * (columnWidth + paddingBetween)} 0)`}
					>
						<text class={'category'} x={`${columnWidth / 2}`} y={'20'} text-anchor={'middle'}
							>{incomeSufficiencyKeyMap[key]}</text
						>
						<text x={`${columnWidth / 2}`} y={'40'} text-anchor={'middle'} font-size = 13px
							>{incomeSufficiencyAmounts[key]}</text
						>
						<g>
							{#each [...Array(Math.floor(incomeSufficiencyAmounts[key] / 5)).keys()] as d, i}
								<circle
									cx={xScale(i % dotsPerRow)}
									cy={yScale((i - (i % dotsPerRow)) / dotsPerRow)}
									id={`dot-${i}`}
									r={4}
									fill={colorScale(ind)}
									transition:fade
								/>
							{/each}
						</g>
					</g>
				{/each}
			</svg>
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
		margin:0;
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
		font-family: 'Nunito', sans-serif;
		width: 200px;
		position: absolute;
	}

	.tooltip-visible {
		font: 25px sans-serif;
		font-family: 'Nunito', sans-serif;
		visibility: visible;
		background-color: #f0dba8;
		border-radius: 10px;
		width: 200px;
		color: black;
		position: absolute;
		padding: 10px;
	}


	.category {
		font-weight: 600;
		font-size: 0.9em;
	}
	
	.current-country {
		padding: .5em;
	}

	.visualization {
		padding:1em;
	}


</style>
