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

	// for back-associating with indices
	let backIncomeSufficiencyAssocs = {
		'enough': 1,
		'almostEnough': 2,
		'sometimesEnough': 3,
		'rarelyEnough': 4,
		'insufficient': 5
	}
	let maxVal = 0;



	// overall chart width, height, and paddings
	let chartWidth = 840; // 800
	let chartHeight = 560; // 1000
	const paddings = {
		top: 10,
		left: 10,
		right: 10,
		bottom: 50
	};
	const paddingBetween = 12; // padding between each column

	// individual column width
	let numCategories = 5;
	let columnWidth =
		(chartWidth - paddings.right - paddings.left - 4 * paddingBetween) / numCategories;

	const dotsPerRow = 25; // 15

	// set scaling variables
	let xScale = scaleLinear()
		.domain([0, dotsPerRow - 1])
		.range([0, columnWidth]); // do positioning of svg's after xScale calculated
	let yScale = scaleLinear() // everything scales full height
		.domain([0, (maxVal - (maxVal % dotsPerRow)) / dotsPerRow])
		.range([paddings.bottom, chartHeight - paddings.top]);
	let colorScale = scaleOrdinal()
		.domain([0, 1, 2, 3, 4])
		.range(['#CCF3FF', '#B9E1FF', '#629FD5', '#1C52A3', '#0E244F']);

	// load data and partition into income sufficiency values
	onMount(async () => {
		// TODO: update once pulled into main
		data = await d3.csv("https://raw.githubusercontent.com/lylakirati/MigrationMDT/grace-styling/data/income_sufficiency.csv"); 
		
		for (let d of data) {
			incomeSufficiencyAmounts[incomeSufficiencyAssocs[+d.income_sufficiency_6m]] += +d.number
		}
		// for (const key of Object.keys(incomeSufficiencyAmounts)) {
		// 	incomeSufficiencyAmounts[key] = incomeSufficiency[key].length;
		// }
		const amounts = Object.keys(incomeSufficiencyAmounts).map(
			(key) => incomeSufficiencyAmounts[key]
		);
		maxVal = Math.max(...amounts);
		xScale = scaleLinear()
			.domain([0, dotsPerRow - 1])
			.range([0, columnWidth]); // do positioning of svg's after xScale calculated
		yScale = scaleLinear() // everything scales full height
			.domain([0, (maxVal - (maxVal % dotsPerRow)) / dotsPerRow])
			.range([paddings.bottom, chartHeight - paddings.top]);
		updateCircles();
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
		// for (const key of Object.keys(incomeSufficiency))
		// {
		// 	incomeSufficiency[key] = [];
		// }
		// for (let d of data) {
		// 	incomeSufficiency[incomeSufficiencyAssocs[+d['income_sufficiency_6m']]].push(d);
		// }
		// console.log(incomeSufficiency);
		Object.keys(incomeSufficiencyAmounts).map((key) => incomeSufficiencyAmounts[key] = 0)
		for (let d of data) {
			const curKey = incomeSufficiencyAssocs[+d['income_sufficiency_6m']];
			incomeSufficiencyAmounts[curKey] += +d.number;
		}
		updateCircles();
	}


	function updateCircles() {
		for(const key of Object.keys(incomeSufficiencyAmounts)) {
			const svg = d3.select(`.${key}`).select('.dots');
			const curData = [...Array(incomeSufficiencyAmounts[key]).keys()];
			svg
				.selectAll('circle')
				.data(curData)
				.join(
					enter => enter.append('circle')
									.attr('cx', (d, i) => xScale(i % dotsPerRow))
									.attr('cy',(d, i) => yScale((i - i % dotsPerRow)/ dotsPerRow) ),
					update => update,
					exit => exit
								.transition()
								.duration(550)
								.ease(d3.easeQuadInOut)
								.style('opacity', 0)
								.remove()
				)
				.attr('cx', (d, i) => xScale(i % dotsPerRow))
				.attr('cy',(d, i) => yScale((i - i % dotsPerRow)/ dotsPerRow) )
				.attr('r', 3)
				.attr('fill', (d) => colorScale(backIncomeSufficiencyAssocs[key] - 1))
				.transition()
				.duration(550)
				.ease(d3.easeQuadInOut)
				.attr('opacity', 1)
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


	
		

</script>

<main>
	
	<label class="selections">
		Choose a country:
		<button id = "button-all" class = "active" on:click={() => updateCountry('all')}> All Countries </button>
		{#each Object.keys(countryMap) as country}
			<button id = "button-{country}" on:click={() => updateCountry(country)}> {countryMap[country]} </button>
		{/each}
	</label>
	

	<h2 class={"current-country"}>Levels of income sufficiency for households in <b>{countryToggle ? countryMap[curCountry] : 'all three countries'}</b></h2>

	<div>One <svg width={10} height={10}><circle r={3} cx={5} cy={5}></circle></svg>  represents 1 household</div>
	<div class="visualization">
		<svg width={chartWidth} height={chartHeight} transition:fade>
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
					<g class="dots">
						<!-- {#each [...Array(Math.floor(incomeSufficiencyAmounts[key])).keys()] as d, i}
							<circle
								cx={xScale(i % dotsPerRow)}
								cy={yScale((i - (i % dotsPerRow)) / dotsPerRow)}
								id={`dot-${i}`}
								r={3}
								fill={colorScale(ind)}
								transition:fade
							/>
						{/each} -->
					</g>
				</g>
			{/each}
		</svg>
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
		padding:0;
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
		padding:0;
	}

	h2 {
		padding:0;
		margin:0;
	}

	b {
		color: #B990EC;
	}
</style>
