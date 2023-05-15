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
    let incomeDataMap = {};
    let incomeZipCodeArray = [];
    let convertedIncome;
    let remitMultiplier;

    let countryMap = {
		SLV: 'El Salvador',
		GT: 'Guatemala',
		HND: 'Honduras'
	};

    let countryInfo = {
        GT: {
            avgIncomeUSD: 42.97,
            avgNoRemitUSD: 35.65,
            avgWithRemitUSD: 69.36,
            currency: "Quetzals",
            exchangeRate: 0.13,
            percentRemittances: 0.217,
            avgRemittancesUSD: 123.052097,
            countryIncomeUSD: 411.66, 
            remitMultiplier: 69.36 / 35.65,
        },
        HND: {
            avgIncomeUSD: 17.21,
            avgNoRemitUSD: 10.58,
            avgWithRemitUSD: 32.17,
            currency: "Lempira",
            exchangeRate: 0.041,
            percentRemittances: 0.289,
            avgRemittancesUSD: 81.108036,
            countryIncomeUSD: 207.50, 
            remitMultiplier: 32.17 / 10.58
        },
        SLV: {
            avgIncomeUSD: 276.61,
            avgNoRemitUSD: 257.04,
            avgWithRemitUSD: 310.00,
            currency: "US Dollars",
            exchangeRate: 1,
            percentRemittances: 0.363,
            avgRemittancesUSD: 147.673111,
            countryIncomeUSD: 355.00, 
            remitMultiplier: 310.00 / 257.04
        }
    };

    // Scale survey income using median income from selected country and median income from target zip code
    // Country: Either 'GT', 'HND', or 'SLV'
    // Zip Code: 5-digit number format
    function convertIncome(country, zipCode) {
        let zipCodeIncome = incomeDataMap[zipCode]?.med_income_monthly
        if(zipCodeIncome === undefined) {
            return undefined; // zip code not in system
        }
        const scaleFactor = zipCodeIncome/countryInfo[country].countryIncomeUSD;
        return countryInfo[country].avgNoRemitUSD * scaleFactor;
    }

    onMount(async () => {
		let data = await d3.csv("https://raw.githubusercontent.com/lylakirati/MigrationMDT/main/mdt/src/data/uszips.csv"); 
        let incomeData = await d3.csv("https://raw.githubusercontent.com/lylakirati/MigrationMDT/main/data/med_income_data.csv");

		for (let d of data) {
			zipCodeData[d.zip] = {zip: d.zip, city: d.city}; // create map
		}
        for (let d of incomeData)
        {
            incomeDataMap[d.zip_code] = {zip: d.zip_code, med_income_monthly: d.med_income_12m !== '-' ? d.med_income_12m / 12 : undefined};
        }
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
        // console.log(userZipCode);
        userZipCodeInfo = zipCodeData[userZipCode]; // TODO: assumes valid zip code for now, fix later
        avgIncomeUSD = countryInfo[country].avgNoRemitUSD;
        currency = countryInfo[country].currency;
        exchangeRate = countryInfo[country].exchangeRate;
        remitMultiplier = countryInfo[country].remitMultiplier;
        if(userZipCodeInfo === undefined)
        {
            alert("Please input a valid zip code");
        }
        else {
            convertedIncome = convertIncome(country, userZipCode); 
            if(convertedIncome === undefined) {
                alert("Sorry, this zip code is currently not in our income system. Please input another zip code.");
            }
            else {
                formSubmitted = true;
                window.scrollBy({
                    top: 500,
                    left:0,
                    behavior: 'smooth'
                });
            }
        }
        

	}

    $: multiplier = 1;
    
    $: budgetAllowance = Math.round((convertedIncome ?? 0) * multiplier); // calculate this logic later
    let allocation = {
        food: 0,
        housing: 0,
        utilities: 0, 
        communication: 0,
        transportation: 0,
        education: 0,
        // not implemented for now:
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

    $: budgetAllowanceWithRemit = Math.round((convertedIncome ?? 0) * (1 + remitMultiplier) * multiplier); // calculate this logic later
    let allocationWithRemit = {
        food: 0,
        housing: 0,
        utilities: 0, 
        communication: 0,
        transportation: 0,
        education: 0,
        // not implemented for now:
       
        // hygieneHousehold: 0,
        // miscExpenses: 0,
        // healthCareMedical: 0,
        // clothesShoes: 0,
        // 
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
        window.scrollBy({
            top: 150,
            left:0,
            behavior: 'smooth'
        })
    }

   
</script>
<main>
    <h2>How could remittances affect your spending?</h2>
    {#if loaded}
    <form on:submit|preventDefault={handleSubmit}>
        <!-- <input id="country" name="country" type="text"> -->
        <div>
            <label for="zip">Please input your zip code:</label>
            <input id="zip" name="zip" type="text" pattern="[0-9]*" required maxLength=5>
            <label for="zip">(Your result will not be stored)</label>
        </div>
        
        <div>
            <label for="country">Select a country to look at:</label>
            <select name="country" id="country" required>
                <option value="SLV">El Salvador</option>
                <option value="GT">Guatemala</option>
                <option value="HND">Honduras</option>
            </select>
        </div>

        <input class="submit" type="submit" value="Go">
    </form>
    {#if formSubmitted}
        <div class="intro">
            <p>
                The average monthly income of survey responses from {countryMap[country]} that do not receive remittances
                is <b>{(avgIncomeUSD/exchangeRate).toFixed(2)} {currency}</b>{#if country !== 'SLV'}
                    , or <b>${avgIncomeUSD.toFixed(2)} US Dollars</b> per month
                {/if}.
            </p>
            <p>
                This is comparable to living on <b>${convertedIncome.toFixed(2)}</b> per month in {userZipCodeInfo.city}. How would you budget
                your monthly allowances with this income?
            </p>
            <!-- <p>
                Input either dollar amounts or percentages, and the rest will be automatically converted.
            </p> -->
        </div>
        <div class="interactive" id="1">
            <h2 class="budget">$ {remainingBudget}</h2>
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
                <Category title={"TRANSPORTATION"} bind:dollars={allocation.transportation} bind:maxValue={budgetAllowance}/>
                <Category title={"EDUCATION"} bind:dollars={allocation.education} bind:maxValue={budgetAllowance}/>
            </div>
            {#if !initialSubmitted}
                <button on:click={submitInitial}>I'm done</button>
            {/if}
        </div>

        {#if initialSubmitted}
            <div class="remittances">
                <p>
                    <b>{countryInfo[country].percentRemittances * 100}%</b> of immigrants to the United States from {countryMap[country]} send remittances 
                    back to their home country.
                </p>
                <p>
                    The average amount of remittances sent from survey respondents per shipment is <b>{(countryInfo[country].avgRemittancesUSD / exchangeRate).toFixed(2)} {currency}</b>{#if country !== 'SLV'}
                    , or <b>${countryInfo[country].avgRemittancesUSD.toFixed(2)} US Dollars</b> per month
                {/if}. 
                </p>
                <p>
                    This provides approximately a <b>{(remitMultiplier * 100).toFixed(2)}</b>% increase in monthly income, or <b>${budgetAllowanceWithRemit}</b> 
                    to live in your city, {userZipCodeInfo.city}.
                </p>
                <p>
                    How would you change your spending with the extra money?
                </p>
            </div>
            <div class="interactive" id="2">
                <h2 class="budget">$ {remainingBudgetWithRemit}</h2>
                <button on:click={resetAmountsWithRemit}>Reset</button>
                <div class="categories">
                    <Category title={"FOOD"} bind:dollars={allocationWithRemit.food} bind:maxValue={budgetAllowanceWithRemit}/>
                    <Category title={"HOUSING"} bind:dollars={allocationWithRemit.housing} bind:maxValue={budgetAllowanceWithRemit}/>
                    <Category title={"UTILITIES"} bind:dollars={allocationWithRemit.utilities} bind:maxValue={budgetAllowanceWithRemit}/>
                    <Category title={"COMMUNICATIONS"} bind:dollars={allocationWithRemit.communication} bind:maxValue={budgetAllowanceWithRemit}/>
                    <Category title={"TRANSPORTATION"} bind:dollars={allocationWithRemit.transportation} bind:maxValue={budgetAllowanceWithRemit}/>
                    <Category title={"EDUCATION"} bind:dollars={allocationWithRemit.education} bind:maxValue={budgetAllowanceWithRemit}/>
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
        text-align:center;
        align-items:center;
        scroll-behavior: smooth;
    }
    .categories {
        
        display:flex;
        flex-direction:row;
        flex-wrap:wrap;
        align-items:space-between;
        justify-content:center;
        gap:3em;
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

    h2 {
        font-size:35px;
        padding: 0 2em;
    }

    .submit  {
        font-family: 'Nunito', sans-serif;
        border: 1px solid black;
        padding: 2px 2em;
        border-radius: 1em;
        background: var(--button-background);
    }


    .submit:hover {
        filter: brightness(95%);
    }

    .intro {
        text-align:left;
        padding: 2em 10em;
    }

    .remittances {
        text-align:left;
        padding: 2em 10em;
    }

    form {
        display:flex;
        flex-direction:column;
        width: fit-content;
        align-items:center;
        justify-content:center;
    }

    select {
        font-family: 'Nunito', sans-serif;
    }

    option {
        font-family: 'Nunito', sans-serif;
    }
    
</style>