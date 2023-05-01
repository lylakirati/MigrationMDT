<script>
	import IncomeSufficiency from './IncomeSufficiency.svelte';
    import scrollama from "scrollama";
    import * as d3 from "d3";
    import {onMount} from "svelte";

    let main2;
    let scrolly2;
    let figure2;
    let article2;
    let step2;

    // initialize the scrollama
    let scroller2 = scrollama();

    // generic window resize listener event
    function handleResize2() {
        // 1. update height of step elements
        let stepH = Math.floor(window.innerHeight * 0.75);
        step2.style("height", stepH + "px");

        console.log("resized");

        let figureHeight = window.innerHeight - 200;
        let figureMarginTop = (window.innerHeight - figureHeight) / 2;

        figure2
            .style("height", figureHeight + "px")
            .style("top", figureMarginTop + "px");

        // 3. tell scrollama to update new element dimensions
        scroller2.resize();
    }

    // scrollama event handlers
    function handleStepEnter2(response) {
        console.log("enter", response);
        step2.classed("is-active", function(d, i) {
          return i === response.index;
        });
        var val = d3.select(response.element).attr("data-step");
        // update graphic based on step
        figure2.select("p").text(response.index + 1);
        figure2.classed("is-fixed", true);
        figure2.classed("is-bottom", false);
    }

    function handleStepExit2(response) {
        console.log("exit", response);
        d3.select(response.element).classed("is-active", false);
    }

    onMount(() =>{
        // main2 = d3.select("main");
		scrolly2 = d3.select("#scrolly2");
		figure2 = scrolly2.select("figure");
	    article2 = scrolly2.select("article");
    	step2 = article2.selectAll(".step");

        console.log(step2);

        // 1. force a resize on load to ensure proper dimensions are sent to scrollama
        
        handleResize2();
        
        // 2. setup the scroller passing options
        // 		this will also initialize trigger observations
        // 3. bind scrollama event handlers (this can be chained like below)
        scroller2
            .setup({
                container:".income-sufficiency",
                step: "#scrolly2 article .step",
                offset: 0.67,
                debug: false
            })
            .onStepEnter(handleStepEnter2)
            .onStepExit(handleStepExit2);
    }

    );

    // kick things off
    // init();
</script>

<section id="scrolly2">
    <figure class="income-suffiency-figure">
        <IncomeSufficiency/>
    </figure>
    <article class="income-sufficiency-text">
        <div class="step" data-step="1">
            <p>
                The fact that the top five reasons for migrating externally are related to finances and money 
                is also reflected in the economic conditions of the people that were surveyed. Overall,
                the most amount of people indicated their current income levels were insufficient. 
            </p>
            
        </div>
        <div class="step" data-step="2">
            <p>
                To Be Expanded Upon
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
    .income-sufficiency {
        margin:0;
    }

    #scrolly2 {
        position: relative;
        display: -webkit-box;
        display: -ms-flexbox;
        display: flex;
        /* background-color: #f3f3f3; */
        padding: 1rem;
    }

    #scrolly2>* {
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

</style>