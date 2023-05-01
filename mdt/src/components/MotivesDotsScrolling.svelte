<script>
    import scrollama from "scrollama";
    import * as d3 from "d3";
    import {onMount} from "svelte";
    import DotGraph from "./DotGraph.svelte";
    import IncomeSufficiency from "./IncomeSufficiency.svelte"

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
        var stepH = Math.floor(window.innerHeight * 0.75);
        step.style("height", stepH + "px");

        var figureHeight = window.innerHeight - 200;
        var figureMarginTop = (window.innerHeight - figureHeight) / 2;

        figure
            .style("height", figureHeight + "px")
            .style("top", figureMarginTop + "px");

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

        console.log(step);

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
            
        <figure class="migration-motives">
            <DotGraph state={index}/>
            
        </figure>
        
        <article>
            <div class="step" data-step="1">
               <p>
                    The World Food Programme surveyed almost 5000 people from Honduras, Guatemala, and El Salvador,
                    collecting data about their economic and living situations, as well as their thoughts towards migration.
                    We're currently looking at all the survey responses that indicated either a desire to migrate externally 
                    (to another country) or a desire not to, if given the chance to do so.
                </p>
            </div>
            <div class="step" data-step="2">
                <p>
                    When separated into their respective categories, we see that while many people do want to migrate 
                    if given a chance to do so, more people actually would prefer to stay in their home country.
                </p>
            </div>
            <div class="step" data-step="3">
                <p>Since we're looking at those who want to migrate though, we will focus on them now.</p>
            </div>
            <div class="step" data-step="4">
                <p>
                    Poverty, food insecurity, and lack of economic opportunities for sustainable livelihoods have 
                    been cited as the top factors behind migration. According to the survey conducted by the World Food 
                    Programme in 2021, more than <b>80 percent</b> of households reported that they want to emigrate in search of 
                    employment opportunities. A large number of people also lack money to secure food and other basic needs, 
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

    figure.is-focused {
        position: -webkit-sticky;
        position: sticky;
    }

</style>