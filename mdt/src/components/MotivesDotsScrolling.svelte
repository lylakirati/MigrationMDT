<script>
    import scrollama from "scrollama";
    import * as d3 from "d3";
    import {onMount} from "svelte";
    import DotGraph from "./DotGraph.svelte";
    // import IncomeSufficiency from "./IncomeSufficiency.svelte"

    let main;
    let scrolly;
    let figure;
    let article;
    let step;

    let index = 0;

    let figureWidth = 800; // responsive, potentially for the future
    // initialize the scrollama
    let scroller = scrollama();

    // generic window resize listener event
    function handleResize() {
        // 1. update height of step elements
        var stepH = Math.floor(window.innerHeight);
        step.style("height", stepH + "px");

        var figureHeight = window.innerHeight - 200;
        var figureMarginTop = (window.innerHeight - figureHeight) / 2;
        figure
            .style("height", figureHeight + "px")
            .style("top", figureMarginTop + "px")
            // .style("left", (window.innerWidth - figureWidth)/2 +"px");

        // 3. tell scrollama to update new element dimensions
        scroller.resize();
    }

    // scrollama event handlers
    function handleStepEnter(response) {
        console.log("enter", response);
        step.classed("is-active", function(d, i) {
          return i === response.index;
        });
        index = response.index;
        var val = d3.select(response.element).attr("data-step");
        // update graphic based on step
        figure.select("p").text(response.index + 1);
    }

    function handleStepExit(response) {
        console.log("exit", response);
        d3.select(response.element).classed("is-active", false);
    }

    onMount(() =>{
        main = d3.select("main");
		scrolly = main.select("#scrolly");
		figure = scrolly.select("figure");
	    article = scrolly.select("article");
    	step = article.selectAll(".step");
        // 1. force a resize on load to ensure proper dimensions are sent to scrollama
        
        handleResize();
        
        // 2. setup the scroller passing options
        // 		this will also initialize trigger observations
        // 3. bind scrollama event handlers (this can be chained like below)
        scroller
            .setup({
                step: "#scrolly article .step",
                offset: 0.33,
                debug: false
            })
            .onStepEnter(handleStepEnter)
            .onStepExit(handleStepExit);
        }
    );

    // kick things off
    // init();
</script>


    <section id="scrolly" bind:this={scrolly}>
            
        <figure class="migration-motives" >
            <DotGraph bind:chartWidth={figureWidth} state={index}/>
            
        </figure>
        
        <article>
            <div class="step" data-step="1">
               <p>
                    The World Food Programme surveyed almost 5,000 households from Honduras, Guatemala, and El Salvador,
                    collecting data about their economic and living situations, as well as their perspectives on migration.
                </p>
            </div>
            <div class="step" data-step="1">
                <p>
                    Each dot represents 1 surveyed household.
                </p>
             </div>
            <div class="step" data-step="2">
                <p>
                    When asked about their intention to move abroad, a large group of them responded 
                    preferring to stay in their home country, citing family separation and rootedness 
                    as main factors. However, <b>over 43% say they do want to migrate if given a chance to do so</b>.
                </p>
            </div>
            <div class="step" data-step="3" id="special">
                <p><b>So what drives these Central Americans to migrate?</b></p>
            </div>
            <div class="step" data-step="4">
                <p>
                    <b>Poverty, food insecurity, and lack of economic opportunities</b>
                    for sustainable livelihoods have 
                    been cited as the top factors behind migration.
                </p>
            </div>
            <div class="step" data-step="5">
                <p>
                    More than <b>80 percent</b>
                    reported that they want to move abroad in search of 
                    <b>employment opportunities</b>. 
                    A large number of households also <b>lack money to secure food and other basic needs</b>, 
                    which motivates them to find jobs in other countries and send money home. 
                </p>
            </div>
        </article>
    </section>

    <section id="outro"></section>

<style>
    
    #scrolly {
        position: relative;
        display: -webkit-box;
        display: -ms-flexbox;
        display: flex;
        flex-direction:column;
        align-items:center;
        justify-content:center;
        /* background-color: #f3f3f3; */
        padding: 1rem;
    }

    #scrolly>* {
        -webkit-box-flex: 1;
        -ms-flex: 1;
        flex: 1;
    }

    section {
        padding:2em 0;
    }

    article {
        position: relative;
        padding: 20 5rem;
        max-width: 30rem;
        /* right: 22rem; */
    }
    
    article p {
        background-color: #FFFFFF;
        padding: 10 5rem;
        box-shadow: 3px 3px 3px #BEBEBE;
    }

    figure {
        position: -webkit-sticky;
        position: sticky;
        width: 80%;
        height: 80vh;
        /* margin-left: 6em; */
        -webkit-transform: translate3d(0, 0, 0);
        -moz-transform: translate3d(0, 0, 0);
        transform: translate3d(0, 0, 0);
        /* background-color: #8a8a8a; */
        z-index: 0;
    }

    .step:last-child {
        margin-bottom: 20em;
    }

    #special {
        text-align:center;
    }

</style>