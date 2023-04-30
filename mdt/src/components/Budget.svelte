<!-- Todo: Come up with a better name later -->
<!-- The Budgeting Interactive Visualization  -->
<script>
	import Category from './Category.svelte';
    import * as d3 from 'd3';
	import { onMount } from 'svelte';

    // whether loaded or not
    let loaded = false;

    // Form data
	let formSubmitted = false;
    let userZipCode = ""; // user zip code
    let userZipCodeInfo; // info from user zip code
    let country = "";
    let avgIncomeUSD, currency, exchangeRate;
    let zipCodeData = {};

    let countryMap = {
		SLV: 'El Salvador',
		GT: 'Guatemala',
		HND: 'Honduras'
	};

    let countryInfo = {
        GT: {
            avgIncomeUSD: 42.97,
            currency: "Quetzals",
            exchangeRate:0.13
        },
        HND: {
            avgIncomeUSD: 17.21,
            currency: "Lempira",
            exchangeRate:0.041
        },
        SLV: {
            avgIncomeUSD: 276.61,
            currency: "Dollars",
            exchangeRate: 1
        }
    };

    onMount(async () => {
		let data = await d3.csv('../../src/data/uszips.csv'); 

		for (let d of data) {
			zipCodeData[d.zip] = d; // create map
		}

        console.log(zipCodeData['95070']);
        loaded = true;
	});

    // From https://svelte.dev/repl/74685aa8b4374c4c8f395ce643fee7b6?version=3.48.0 
    const handleSubmit = e => {
		// getting the action url
		const ACTION_URL = e.target.action;

		// get the form fields data and convert it to URLSearchParams
		const formData = new FormData(e.target);
		for (let field of formData) {
			const [key, value] = field;
            // please find a better way of doing this
            if(key === 'zip') {
                userZipCode = value;
            }
            else {
                country = value;
            }
		}
        console.log(userZipCode);
        userZipCodeInfo = zipCodeData[userZipCode]; // TODO: assumes valid zip code for now, fix later
        avgIncomeUSD = countryInfo[country].avgIncomeUSD;
        currency = countryInfo[country].currency;
        exchangeRate = countryInfo[country].exchangeRate;
        formSubmitted = true;
	}


    let budgetAllowance = 2000; // calculate this logic later
    let allocation = {
        food: 0,
        housing: 0,
        utilities: 0, 
        // not implemented for now:
        // communication: 0,
        // transportation: 0,
        // hygieneHousehold: 0,
        // miscExpenses: 0,
        // healthCareMedical: 0,
        // clothesShoes: 0,
        // education: 0,
        // debt: 0,
        // socialEvents: 0,
        // productiveSupplies: 0,
        // savings: 0,
        // constructionRepair: 0
    }

    $: remainingBudget = budgetAllowance - Object.keys(allocation).reduce((sum, key) => allocation[key] + sum, 0)
</script>
<main>
    {#if loaded}
    <form on:submit|preventDefault={handleSubmit}>
        <!-- <input id="country" name="country" type="text"> -->
        <label for="zip">Please Input Your Zip Code:</label>
        <input id="zip" name="zip" type="text" pattern="[0-9]*" required maxLength=5>

        <label for="country">Please Select a Country to Look at:</label>
        <select name="country" id="country" required>
            <option value="GT">Guatemala</option>
            <option value="HND">Honduras</option>
            <option value="SLV">El Salvador</option>
        </select>

        <input type="submit" value="Go">
    </form>
    {#if formSubmitted}
        <div class="intro">
            <p>
                The average monthly income of survey responses from {countryMap[country]} is {(avgIncomeUSD/exchangeRate).toFixed(2)} {currency}, 
                or {avgIncomeUSD} US dollars per month. 
            </p>
            <p>
                This is comparable to living on <b>figure this out</b> per month in {userZipCodeInfo.city}. How would you budget
                your monthly allowances with this income?
            </p>
            <p>
                Input either dollar amounts or percentages, and the rest will be automatically converted.
            </p>
        </div>
        <div class="interactive">
            <h2>{remainingBudget} $</h2>
            <div class="categories">
                <Category title={"FOOD"} bind:dollars={allocation.food} bind:maxValue={remainingBudget}/>
                <Category title={"HOUSING"} bind:dollars={allocation.housing} bind:maxValue={remainingBudget}/>
                <Category title={"UTILITIES"} bind:dollars={allocation.utilities} bind:maxValue={remainingBudget}/>
            </div>
        </div>
    {/if}
    {:else} 
    <div>Loading</div>
    {/if}
</main>

<style>
    main {
        display: flex;
        flex-direction:column;
        gap: 2em;
    }
    .categories {
        display:flex;
        flex-direction:row-wrap;
        align-items:center;
        justify-content:center;
        gap:2em;
    }
</style>