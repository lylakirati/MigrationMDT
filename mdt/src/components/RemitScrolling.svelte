<script>
	import IncomeSufficiency from './IncomeSufficiency.svelte';
    import scrollama from "scrollama";
    import * as d3 from "d3";
    import {onMount} from "svelte";
	import Spending from './Spending.svelte';

    let main3;
    let scrolly3;
    let figure3;
    let article3;
    let step3;

    let index = 0;

    // initialize the scrollama
    let scroller3 = scrollama();

    // generic window resize listener event
    function handleResize3() {
        // 1. update height of step elements
        let stepH = Math.floor(window.innerHeight * 0.75);
        step3.style("height", stepH + "px");

        console.log("resized");

        let figureHeight = window.innerHeight - 100;
        let figureMarginTop = (window.innerHeight - figureHeight) / 3;

        figure3
            .style("height", figureHeight + "px")
            .style("top", figureMarginTop + "px");

        // 3. tell scrollama to update new element dimensions
        scroller3.resize();
    }

    // scrollama event handlers
    function handleStepEnter3(response) {
        console.log("enter", response);
        step3.classed("is-active", function(d, i) {
          return i === response.index;
        });
        // index = response.index;
        var val = d3.select(response.element).attr("data-step");
        // update graphic based on step
        figure3.select("p").text(response.index + 1);
        figure3.classed("is-fixed", true);
        figure3.classed("is-bottom", false);
    }

    function handleStepExit3(response) {
        console.log("exit", response);
        index = response.index;
        d3.select(response.element).classed("is-active", false);
    }

    onMount(() =>{
        // main3 = d3.select("main");
		scrolly3 = d3.select("#scrolly3");
		figure3 = scrolly3.select("figure");
	    article3 = scrolly3.select("article");
    	step3 = article3.selectAll(".step");

        console.log(step3);

        // 1. force a resize on load to ensure proper dimensions are sent to scrollama
        
        handleResize3();
        
        // 3. setup the scroller passing options
        // 		this will also initialize trigger observations
        // 3. bind scrollama event handlers (this can be chained like below)
        scroller3
            .setup({
                container:".income-sufficiency",
                step: "#scrolly3 article .step",
                offset: 1,
                debug: false
            })
            .onStepEnter(handleStepEnter3)
            .onStepExit(handleStepExit3);
    }

    );

    // kick things off
    // init();
</script>

<section id="scrolly3">
    <figure>
        <Spending state = {index}/>
        <!-- <p>0</p> -->
    </figure>
    <article class="income-sufficiency-text">
        <div class="step" data-step="1">
            <p>
                <b>Twelve%</b>
                of the households indicate that they <b>receive financial support from abroad at least once a month</b>.
            </p>
            <p>
                The amount of the remittances is <b>US$152 on average and can go up to US$1,000 per shipment</b>.
            </p>
        </div>
        <div class="step" data-step="2">
            <p>
                This is what the <b>average monthly expenditure of households who receive financial support
                from abroad</b> looks like. They spend approximately US$205 per month. 
            </p>
        </div>
        <div class="step" data-step="3">
            <p>
                Housing costs in the Central American countries are less expensive than in the U.S., but 
               <b>food and drinking water</b> cost more, comprising <b>over 35%</b> of Central American households’ monthly expenditure.
            </p>
        </div>
        <div class="step" data-step="4">
            <p>
                Households who receive no financial support from abroad reported <b>spending less 
                across all categories</b> of monthly expenditures. To afford the more fixed expenses such as 
                rent and transportation, households make <b>greater cuts to spending on food and healthcare</b>.
            </p>
        </div>
        <div class="step" data-step="5">
            <p>
                While the amount of remittances received vary by country and by individual households, 
                the <b>additional source of income</b> plays a substantial role in improving the livelihoods of 
                families that receive them.
            </p>
        </div>
        <!-- <div class="step" data-step="3">
            <p>STEP 3</p>
        </div>
        <div class="step" data-step="4">
            <p>STEP 4</p>
        </div> -->
    </article>
</section>

    


<style>

    #scrolly3 {
        position: relative;
        display: -webkit-box;
        display: -ms-flexbox;
        display: flex;
        /* background-color: #f3f3f3; */
        padding: 1rem;
    }

    #scrolly3>* {
        -webkit-box-flex: 1;
        -ms-flex: 1;
        flex: 1;
    }

    section {
        padding:2em 0;
    }

    article {
        position: relative;
        padding: 0 1rem;
        max-width: 20rem;
    }

    figure {
        position: -webkit-sticky;
        position: sticky;
        width: 100%;
        height:80vh;
        margin: 0;
        -webkit-transform: translate3d(0, 0, 0);
        -moz-transform: translate3d(0, 0, 0);
        transform: translate3d(0, 0, 0);
        /* background-color: #8a8a8a; */
        z-index: 0;
    }

    figure .is-fixed{
        position:sticky;
    }
    figure p {
        text-align: center;
        padding: 1rem;
        position: absolute;
        top: 50%;
        left: 50%;
        -moz-transform: translate(-50%, -50%);
        -webkit-transform: translate(-50%, -50%);
        transform: translate(-50%, -50%);
        font-size: 8rem;
        font-weight: 900;
        /* color: #fff; */
    }

    .step {
        text-align:left;
        margin: 0 auto 2rem auto;
        border-left: 2px solid var(--primary-color);
    }

    .step:last-child {
        margin-bottom: 20em;
    }

    .step.is-active {
        background-color: goldenrod;
        color: #3b3b3b;
    }
</style>