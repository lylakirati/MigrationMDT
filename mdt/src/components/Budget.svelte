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

    $: multiplier = 1;
    console.log(multiplier);
    $: budgetAllowance = Math.floor(400 * multiplier); // calculate this logic later
    let allocation = {
        food: 0,
        housing: 0,
        utilities: 0, 
        communication: 0
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

    $: budgetAllowanceWithRemit = Math.floor(600 * multiplier); // calculate this logic later
    let allocationWithRemit = {
        food: 0,
        housing: 0,
        utilities: 0, 
        communication: 0
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
    $: remainingBudgetWithRemit = budgetAllowanceWithRemit - Object.keys(allocationWithRemit).reduce((sum, key) => allocationWithRemit[key] + sum, 0)

    let initialSubmitted = false;

    function resetAmounts() {
        Object.keys(allocation).map((key) => allocation[key] = 0)
        multiplier = 1;
    }
    function resetAmountsWithRemit() {
        Object.keys(allocationWithRemit).map((key) => allocation[key] = 0)
    }
    function submitInitial() {
        initialSubmitted = true;
    }

   
</script>
<main>
    <h2>How would your spending be affected by remittances?</h2>
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
                This is comparable to living on <b>400 (placeholder)</b> per month in {userZipCodeInfo.city}. How would you budget
                your monthly allowances with this income?
            </p>
            <p>
                Input either dollar amounts or percentages, and the rest will be automatically converted.
            </p>
        </div>
        <div class="interactive" id="1">
            <h2 class="budget">{remainingBudget} $</h2>
            <div class="options">
                <button on:click={resetAmounts}>Reset All Values</button>
                {#if initialSubmitted}
                    <div>Experiment with different incomes</div>
                    <input type="range" min="0.1" max="5" class="slider" step="0.1" bind:value={multiplier}>
                {/if}
            </div>
            <div class="categories">
                <Category title={"FOOD"} bind:dollars={allocation.food} bind:maxValue={budgetAllowance}/>
                <Category title={"HOUSING"} bind:dollars={allocation.housing} bind:maxValue={budgetAllowance}/>
                <Category title={"UTILITIES"} bind:dollars={allocation.utilities} bind:maxValue={budgetAllowance}/>
                <Category title={"COMMUNICATIONS"} bind:dollars={allocation.communication} bind:maxValue={budgetAllowance}/>
            </div>
            {#if !initialSubmitted}
                <button on:click={submitInitial}>I'm done</button>
            {/if}
        </div>

        {#if initialSubmitted}
            <div class="remittances">
                <p><b>TBD</b>% of immigrants to the United States from {countryMap[country]} send remittances 
                back to their home country.</p>
                <p>
                    The median amount of remittances sent from survey respondents is $<b>TBD</b> per month, or
                    <b>convert to local currency</b>
                </p>
                <p>
                    This provides a <b>TBD</b>% increase in budget, or $ <b>TBD</b> 
                    to live in your city, {userZipCodeInfo.city}.
                </p>
                <p>
                    How would you change your spending with this additional budget?
                </p>
            </div>
            <div class="interactive" id="2">
                <h2 class="budget">{remainingBudgetWithRemit} $</h2>
                <button on:click={resetAmountsWithRemit}>Reset</button>
                <div class="categories">
                    <Category title={"FOOD"} bind:dollars={allocationWithRemit.food} bind:maxValue={budgetAllowanceWithRemit}/>
                    <Category title={"HOUSING"} bind:dollars={allocationWithRemit.housing} bind:maxValue={budgetAllowanceWithRemit}/>
                    <Category title={"UTILITIES"} bind:dollars={allocationWithRemit.utilities} bind:maxValue={budgetAllowanceWithRemit}/>
                    <Category title={"COMMUNICATIONS"} bind:dollars={allocationWithRemit.communication} bind:maxValue={budgetAllowanceWithRemit}/>
                </div>
            </div>
        {/if}
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
        min-height:100vh;
        padding-bottom:2em;
    }
    .categories {
        display:flex;
        flex-direction:row-wrap;
        align-items:space-between;
        justify-content:center;
        gap:2em;
    }

    .interactive {
        display:flex;
        flex-direction:column;
        align-items:center;
        justify-content:center;
        gap:2em;
    }

    .interactive .budget {
        margin:0;
    }
</style>